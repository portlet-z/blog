用来进行线程同步协作，等待所有线程完成倒计时

其中构造函数参数用来初始化等待计数值，await()用来等待计数归零，countDown()用来让计数减一

```java
@Slf4j
public class TestCountDownLatch {
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(4);
        CountDownLatch countDownLatch = new CountDownLatch(3);
        service.submit(() -> {
            log.debug("begin...");
            sleep(1000);
            countDownLatch.countDown();
            log.debug("end... {}", countDownLatch.getCount());
        });
        service.submit(() -> {
            log.debug("begin...");
            sleep(1500);
            countDownLatch.countDown();
            log.debug("end... {}", countDownLatch.getCount());
        });
        service.submit(() -> {
            log.debug("begin...");
            sleep(2000);
            countDownLatch.countDown();
            log.debug("end... {}", countDownLatch.getCount());
        });
        service.submit(() -> {
            try {
                log.debug("waiting...");
                countDownLatch.await();
                log.debug("wait end... {}");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
    private static void sleep(long timeout) {
        try {
            Thread.sleep(timeout);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
@Slf4j
public class TestCountDownLatch {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(10);
        CountDownLatch countDownLatch = new CountDownLatch(10);
        SecureRandom random = new SecureRandom();
        String[] all = new String[10];
        for (int j = 0; j < 10; j++) {
            int k = j;
            service.submit(() -> {
                for (int i = 0; i <= 100; i++) {
                    sleep(random.nextInt(100));
                    all[k] = i + "%";
                    System.out.print("\r" + Arrays.toString(all));
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("\n游戏开始");
        service.shutdown();
    }
}
```

