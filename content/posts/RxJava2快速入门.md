---
date: "2018-03-24 17:55:56"
title: RxJava2快速入门
---
# RxJava2快速入门

## 引入依赖

```
compile 'io.reactivex.rxjava2:rxjava:2.0.1'
```

## 写法

### 简单版本

```
	private static void helloSimple() {
        Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                System.out.println("consumer accept is " + s);
            }
        };

        Observable.just("hello world").subscribe(consumer);
	}
```

### 复杂版本

```
	private static void helloComplex() {
        Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                System.out.println("onSubscribe: " + d);
            }

            @Override
            public void onNext(String s) {
                System.out.println("onNext: " + s);
            }

            @Override
            public void onError(Throwable e) {
                System.out.println("onError: " + e);
            }

            @Override
            public void onComplete() {
                System.out.println("onComplete: ");
            }
        };

        Observable.just("Hello world").subscribe(observer);
    }
```

### 变态版本

```
	private static void helloPlus() {
        Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                System.out.println("onSubscribe: " + d);
            }

            @Override
            public void onNext(String s) {
                System.out.println("onNext: " + s);
            }

            @Override
            public void onError(Throwable e) {
                System.out.println("onError: " + e);
            }

            @Override
            public void onComplete() {
                System.out.println("onComplete: ");
            }
        };

        Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                e.onNext("hello world");
                e.onComplete();
            }
        });

        observable.subscribe(observer);
    }
```

## 常用操作符

### filter

你早上去吃早餐，师傅是被观察者，说咱这有"包子", "馒头", "花生", "牛奶", "饺子", "春卷", "油条"，你仔细想了想，发现你是最喜欢饺子的，所以把其他的都排除掉，
于是你就吃到了饺子。

```
	private static void helloFilter() {
        Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                System.out.println("accept: " + s);
            }
        };

        Observable.just("包子", "馒头", "花生", "牛奶", "饺子", "春卷", "油条")
                .filter(new Predicate<String>() {
                    @Override
                    public boolean test(String s) throws Exception {
                        System.out.println("test: " + s);
                        return s.equals("饺子");
                    }
                })
                .subscribe(consumer);
    }
```

### Map

map操作符能够完成数据类型的转换。

将String类型转换为Integer类型。

```
	private static void helloMap() {
        // 观察者观察Integer
        Observer<Integer> observer = new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                System.out.println("onSubscribe: " + d);
            }

            @Override
            public void onNext(Integer s) {
                System.out.println("onNext: " + s);
            }

            @Override
            public void onError(Throwable e) {
                System.out.println("onError: " + e);
            }

            @Override
            public void onComplete() {
                System.out.println("onComplete: ");
            }
        };

        Observable.just("100")
                .map(new Function<String, Integer>() {
                    @Override
                    public Integer apply(String s) throws Exception {
                        return Integer.valueOf(s);
                    }
                })
                .subscribe(observer);
    }
```

### FlatMap

flatmap能够链式地完成数据类型的转换和加工。

遍历一个学校所有班级所有组的所有学生

```
private void flatmapClassToGroupToStudent() {
    Observable.fromIterable(new School().getClasses())
            //输入是Class类型，输出是ObservableSource<Group>类型
            .flatMap(new Function<Class, ObservableSource<Group>>() {
                @Override
                public ObservableSource<Group> apply(Class aClass) throws Exception {
                    Log.d(TAG, "apply: " + aClass.toString());
                    return Observable.fromIterable(aClass.getGroups());
                }
            })
            //输入类型是Group，输出类型是ObservableSource<Student>类型
            .flatMap(new Function<Group, ObservableSource<Student>>() {
                @Override
                public ObservableSource<Student> apply(Group group) throws Exception {
                    Log.d(TAG, "apply: " + group.toString());
                    return Observable.fromIterable(group.getStudents());
                }
            })
            .subscribe(
                    new Observer<Student>() {
                        @Override
                        public void onSubscribe(Disposable d) {
                            Log.d(TAG, "onSubscribe: ");
                        }

                        @Override
                        public void onNext(Student value) {
                            Log.d(TAG, "onNext: " + value.toString());
                        }

                        @Override
                        public void onError(Throwable e) {

                        }

                        @Override
                        public void onComplete() {

                        }
                    });
}

```

# 线程调度

关于RxJava的线程调度，初学者只需要掌握两个api就够够的啦。

## subscribeOn

指定Observable在一个指定的线程调度器上创建。只能指定一次，如果指定多次则以第一次为准

## observeOn

指定在事件传递，转换，加工和最终被观察者接受发生在哪一个线程调度器。可指定多次，每次指定完都在下一步生效。

## 常用线程调度器类型

- Schedulers.single()  单线程调度器，线程可复用
- Schedulers.newThread() 为每个任务创建新的线程
- Schedulers.io() 处理io密集型任务，内部是线程池实现，可自动根据需求增长
- Schedulers.computation() 处理计算任务，如事件循环和回调任务
- AndroidSchedulers.mainThread() Android主线程调度器
