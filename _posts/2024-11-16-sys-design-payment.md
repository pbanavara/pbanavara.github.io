### How to design a payments system for e-commerce?
I was just thinking through some of the past problems that I have solved in my career and retrospecting about how I would solve them today. I was in a group discussion with a bunch of folks discussing a payments system design use case and it brought back memories of how I went about designing the system. Given the proprietary nature of what I did, I want to focus on what I could have done differently or better. 

One aspect I overlooked while discussing the problem was the actual flow of payments in e-commerce. The discussion helped me categorize the problem upfront into either
* A realtime payment system for digital e-commerce (Kindle etc)
* An offline payment system for physical e-commerce. 

The key difference between the two is the ACID requirements of the system. The real-time payment system needs to be ACID compliant, whereas the offline payment system can be eventually consistent, oh well parts of the offline payment system can be eventually consistent.

### Real-time payment system
What are the major components of a real-time payment system?
* A pre-payment processor 
  * Account verification
  * Fraud check
  * Order ID check
* A payment processor
  * Post payment to the gateway/service provider
* A post-payment processor
  * Send order delivery notification

#### Requirements
* The only requirement is for the payment to be processed once and only once for a given order.
* The other requirement could be the ability to rollback the payment for whatever reason (digital download failed etc)
* Typically the scale requirements are in the order of 10 to 100 transactions per second. This is another design hack that I learned. Jump to the scale immediately to understand what design pattern to go after. Like if the requirement is for 1000 transactions per second, then it'll be really hard to design an ACID compliant system unless you go for distributed transactions which are really hard to scale. Only a few sytesms on the planet are capable of handling distributed transactions Google Spanner and AWS Aurora are some of them.
  
#### Design
I don't believe in drawing out the design diagrams for such problems. Talking through code is what I prefer. The talking through code approach has several advantages.
* You can easily upgrade the code to API design. 
* It helps distill thoughts in a concise manner - brevity is the soul of wit.

#### Simple approach
* We need ACID semantics for the payment system. 
```python
def make_payment(customer_id, order_id, payment_id, amount, currency):
    if order.exists(order_id):
        if check_fraud(customer_id, payment_id):
            if send_payment(customer_id, payment_id, amount, currency):
                order.update(order_id, payment_id, status=PAID)
                return True
            else:
                return False
        else:
            return False
    else:
        return False
```
The drawbacks of this approach are really obvious. 
* Very strong coupling of one API service with another service.
* No distinct error handling for different failure types.
* No scope to rollback if the innermost update fails.

A better approach is to use a slightly advanced transaction block to decorate the multiple services.

```python
async def process_payment(customer_id, order_id, payment_id, amount, currency):
    async with transation.atomic():
        verification = await verify_customer(customer_id, payment_id)
        if not verification:
            raise TransactionFailed("Customer verification failed")
        fraud_check = await check_fraud(customer_id, payment_id)
        if not fraud_check:
            raise TransactionFailed("Fraud check failed")
        payment_status = await send_payment(customer_id, payment_id, amount, currency)
        if not payment_status:
            raise TransactionFailed("Payment failed")
        delivery_status = await send_delivery_notification(customer_id, order_id)
        if not delivery_status:
            raise TransactionFailed("Delivery notification failed")
    return True
```
This approach satisfies the ACID requirements of the system, yet is decoupled and has a clear error handling mechanism.
The only drawback here is that this is hard to scale in a language like Python because of the GIL. The only way to scale this is to use multi processing which is expensive. Perhaps the same approach in a language that supprts multi threading like Java could be a better approach. So let's see how a similar approach could be implemented in Java.

```java
@service
@Transactional
public class PaymentService {
    @Autowired
    private CustomerVerificationService customerVerificationService;
    @Autowired
    private FraudCheckService fraudCheckService;
    @Autowired
    private PaymentService paymentService;

    public PaymentResult processPayment(Customer customer, Order order, Payment payment) {
        TransactionStatus status = TransactionStatus.SUCCESS;
        try {
            // start transaction
            status = TransactionManager.getTransaction(new DefaultTransactionDefinition());
            if (status != TransactionStatus.ACTIVE) {
                throw new TransactionFailedException("Transaction failed to start");
            }
            CustomerVerificationResult verificationResult = 
                customerVerificationService.verifyCustomer(customer, payment);
            if (!verificationResult.isSuccess()) {
                throw new TransactionFailedException("Customer verification failed");
            }
            FraudCheckResult fraudCheckResult = fraudCheckService.checkFraud(customer, payment);
            if (!fraudCheckResult) {
                throw new TransactionFailedException("Fraud check failed");
            }
            PaymentResult paymentResult = paymentService.processPayment(customer, order, payment);
            if (!paymentResult.isSuccess()) {
                throw new TransactionFailedException("Payment failed");
            }
            DeliveryNotificationResult deliveryNotificationResult = 
                deliveryNotificationService.sendDeliveryNotification(customer, order);
            if (!deliveryNotificationResult.success) {
                throw new TransactionFailedException("Delivery notification failed");
            }
            TransactionManager.commit(status);
            return PaymentResult.success();
        } catch (Exception e) {
            if (status == TransactionStatus.ACTIVE) {
                TransactionManager.rollback(status);
            }
            throw new TransactionFailedException("Transaction failed");
        }
    }
}

// Inside the client
PaymentService paymentService = new PaymentService();
PaymentResult status = paymentService.processPayment(customer, order, payment);
if (status.isSuccess()) {
    System.out.println("Payment successful");
}
```
So far so good, ACID compliance, ability to rollback, and clear error handling. Let's add multithreading to this to scale better.

```java
@service
public class PaymentService {

    private final ExecutorService executor = Executors.newFixedThreadPool(10);

    public CompletableFuture<PaymentResult> processPayment(Customer customer, Order order, Payment payment) {
        return CompletableFuture.supplyAsync(() -> {
            TransactionStatus status = null;
            try {
                status = TransactionManager.getTransaction(new DefaultTransactionDefinition());
                if (status != TransactionStatus.ACTIVE) {
                    throw new TransactionFailedException("Transaction failed to start");
                }
                CustomerVerificationResult verificationResult = 
                    customerVerificationService.verifyCustomer(customer, payment);
                if (!verificationResult.isSuccess()) {
                    throw new TransactionFailedException("Customer verification failed");
                }
                FraudCheckResult fraudCheckResult = fraudCheckService.checkFraud(customer, payment);
                if (!fraudCheckResult) {
                    throw new TransactionFailedException
                    ("Fraud check failed");
                }
                PaymentResult paymentResult = paymentService.processPayment(customer, order, payment);
                if (!paymentResult.isSuccess()) {
                    throw new TransactionFailedException("Payment failed");
                }
                DeliveryNotificationResult deliveryNotificationResult =
                    deliveryNotificationService.sendDeliveryNotification(customer, order);
                if (!deliveryNotificationResult.success) {
                    throw new TransactionFailedException("Delivery notification failed");
                }
                TransactionManager.commit(status);
                return paymentResult;
            } catch Exception e) {
                if (status == TransactionStatus.ACTIVE) {
                    TransactionManager.rollback(status);
                }
                throw new TransactionFailedException("Transaction failed");
            }
    }
}

// client
List<Stringg> orderIds = Arrays.asList("123", "456", "789");
public List<PaymentResult> processPayments(List<String> orderIds) {
    List <CompletableFuture<PaymentResult>> futures = 
        orderIds.stream()
        .map(orderId => paymentService.processPayment(customer, order, payment))
        .collect(Collectors.toList());
    
    return futures.stream().map(CompletableFuture::join).collect(Collectors.toList());
}
```
Now we are getting somewhere. We have multithreading, ACID semantics and rollback. However, this is still batch processing. We need to be able to process real time payments. So we will modify this to a Queue implememtation.
Some points to consider while implementing a Queue:
* We need to handle backpressure. We need to block the producer if the queue is full.
* We can't just reject messages silently since this is a payment processor. So we need a blocking call.
* OR we need to dynamically resize the queue.

#### Real time ACID transactions using Queues
```java
@service
public class PaymentService {
    private final ExecutorService executor = Executors.newFixedThreadPool(10);
    private final BlockingQueue<PaymentRequest> paymentQueue = new BlockingQueue<>(1000);

    @PostConstruct
    public void start() {
        for (int i = 0; i < 10; i++) {
            executor.submit(this.processPaymentQueue());
        }
    }
    
    public CompletableFuture<PaymentResult> processPayment(Customer customer, Order order, Payment payment) {
        return CompletableFuture.supplyAsync(() -> {
            // Same as above
        }
    }

    private void processPaymentQueue() {
        while (! Thread.currentThread().isInterrupted()) {
            try {
                PaymentRequest request = paymentQueue.take();
                processPayment(request.customer, request.order, request.payment)
                .thenAccept(paymentResult -> notify(paymentResult))
                .exceptionally(e -> {
                    handlePaymentError(e);
                    return null;
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();

            }

        }
    }

    public void queuePayment(Customer customer, Order order, Payment payment) {
        paymentQueue.offer(new PaymentRequest(customer, order, payment));
    }

    public void queuePaymentBatch(List<PaymentRequest> paymentRequests) {
        paymentRequests.forEach(paymentQueue::offer);
    }
}
```
Now we have a real time payment processor. We can scale it horizontally by adding more threads. We need to figure out how to dynamically resize the queue, if at all it is feasible.

I am quite happy with this progression. Now I can think of databases and how to commit these transactions to database, how the database transaction will work at scale etc.


    

