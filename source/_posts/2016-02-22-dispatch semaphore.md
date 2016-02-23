---
layout: post
comments: true
categories: iOS
---

# Dispatch Semaphore

不考虑顺序将所有数据添加到`NSMutableArray`中。

```
dispatch_queue_t queue = diapatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

NSMutableArray *array = [[NSMutableArray alloc] init];

for (int i = 0; i < 100000; i++) {
	dispatch_async(queue, ^{
		[array addobject:@i];
	});
}
```

执行后，由内存错误导致应用程序异常结束的概率很高。所以，此时应使用`Dispatch Semaphore`。

通过`dispatch_semaphore_create`函数生成Dispatch Semaphore

```
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
```

参数表示计数的初始值，`dispatch_semaphore_t`必须通过`dispatch_release`函数释放.

```
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
```

`dispatch_semaphore_wait`函数等待`Dispatch Semaphore`的计量数达到大于或等于1。第二个参数是由`dispatch_time_t`类型值，指定等待时间(DISPATCH_TIME_FOREVER永久等待)。

```
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, lull * NSEC_PER_SEC);

long result = dispatch_semaphore_wait(semaphore, time);

if (result == 0) {
    
    /*
    *   由于semaphore的计数值大于等于1
    *   或者在等待时间内，semaphore的计数值大于等于1
    *   所以semaphore得计数值减去1
    *
    *   可执行需要进行排他控制的处理
    */
    
}
else {
    
    /*
     *   dispatch_semaphore_wait的返回值也为long型。当其返回0时表示在timeout之前，该函数所处的线程被成功唤醒。
     *　　当其返回不为0时，表示timeout发生。
    */
}
```

我们在前面的源码中实际使用Dispatch Semaphore：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);

NSMutableArray *array = [[NSMutableArray alloc] init];

for (int i = 0; i < 10000; i++) {
	dispatch_async(queue, ^{
		dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
		
		[array addObject:[NSNumber numberWithInt:i];
		
		dispatch_semaphore_signal(semaphore);
	});
	
}

/*
 *	如果使用结束，需要如下这样释放：
 * 释放Dispatch Semaphore
 * 
 * dispatch_release(semaphore);
 */
```

## 下面还有一种用法

The question is: How do I wait for an asynchronously dispatched block to finish, in ARC?

SINGLE BLOCK

```
dispatch_semaphore_t sem = dispatch_semaphore_create(0);

[self methodWithABlock:^(id result){
    //put code here
    dispatch_semaphore_signal(sem);
}];

dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
```

MULTIPLE BLOCKS

```
dispatch_semaphore_t sem = dispatch_semaphore_create(0);

[self methodWithABlock:^(id result){
    //put code here
    dispatch_semaphore_signal(sem);
}];

[self methodWithABlock:^(id result){
    //put code here
    dispatch_semaphore_signal(sem);
}];

dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
```

BLOCK IN BLOCK

```
dispatch_semaphore_t sem = dispatch_semaphore_create(0);

[self methodWithABlock:^(id result){
    //put code here
    dispatch_semaphore_signal(sem);
    
    [self methodWithABlock:^(id result){
        //put code here
        dispatch_semaphore_signal(sem);
    }];
}];

dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
```

