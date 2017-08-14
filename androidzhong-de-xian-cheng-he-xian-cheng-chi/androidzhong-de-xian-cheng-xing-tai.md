# AsyncTask详解

## AsyncTask的介绍

**它不过是对线程池和Handler的封装** ；用线程池来处理后台任务，用Handler来处理与UI的交互。

AsyncTask是Android中提供的一个异步操作框架， 在Android还有Thread和Handler来执行异步操作，AsyncTask与它们的区别是:AsyncTask更适合来做耗时比较小的操作，如果是需要一个线程长时间在后台运行，可以使用Executor,ThreadPoolExecutor,FutureTask。

## AsyncTask的用法

### 执行任务

```java
/**
     * 使用AsyncTask的时候必须要实现AsyncTask<Params,Progress,Result>的子类。
     * 第一个参数用于传入到doInBackground方法中
     * 第二个参数是指的耗时操作在后台执行的进度
     * 第三个指的后台操作完成后，返回的结果
     * 如果某个类型不用，在继承的时候直接使用Void类。
     * 四个过程分别代表：
     * onPreExecute: 异步执行前的回调，在UI线程调用，通常用来初始化。
     * doInBackground: 异步执行的内容，在后台线程调用。在执行过程中可以通过publishProgress发起更新回调。
     * onProgressUpdate: 异步执行过程中的通知，它就是publishProgress的回调方法。它的内容是在UI线程中执行。
     * onPostExecute: 异步执行结束后的回调方法，它在UI线程中执行。
     * 可以看到除了doInBackground方法之外其它的回调都是在UI线程中执行的。
     */
    class MyTask extends AsyncTask<String,Void,String>{
        //在UI线程执行，doInBackground()方法前执行，通常用来初始化
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
        }
        //执行耗时操作，在这里可以调用publishProgress()方法来更新进度的操作
        @Override
        protected String doInBackground(String... params) {
            double result,opNum1,opNum2;
            char op = ' ';
            String para = params[0];
            int index = 0;
            for(int i = 0;i < para.length();i++){
                if(!Character.isDigit(para.charAt(i))){
                    index = i;
                    op = para.charAt(i);
                    break;
                }
            }
            opNum1 = Double.parseDouble(para.substring(0,index));
            opNum2 = Double.parseDouble(para.substring(index+1));

            switch (op){
                case '+':result = opNum1 + opNum2;break;
                case '-':result = opNum1 - opNum2;break;
                case '*':result = opNum1 * opNum2;break;
                case '/':result = opNum1 / opNum2;break;
                default:return "error";
            }
            return String.valueOf(result);
        }


        //在UI线程中执行，在doInBackground调用publishProgress(Progress...)后执行
        //用于在前台显示后台进程的完成情况
        @Override
        protected void onProgressUpdate(Void... values) {
            super.onProgressUpdate(values);
        }
        //在耗时操作完成后，触发这个方法，在UI线程中执行，更新UI，告诉用户后台操作已经完成
        @Override
        protected void onPostExecute(String s) {
            super.onPostExecute(s);
        }
    }
```

### 取消任务

可以随时使用`cancel(boolean)`方法来取消一个任务。一个任务被取消后，系统会执行`onCancelled()`作为结果回调接口，而不是使用`onPostExecute(Object)`。可以使用`isCancelled(Object)`方法判断一个任务是否被取消。如果一个任务可能被取消的话，就尽量在`doInBackground()`方法中定期的检查`isCancel()`方法，如果任务被取消可以提早结束任务，节约资源。

### 注意事项

1. `AsyncTask`类必须在主线程中加载。在`Build.VERSION_CODES.JELLY_BEAN`\(API 16\)版本以后自动完成。
2. `AsyncTask`类的实例必须在UI线程中被创建\(Android5.1以前\)。
3. `execute`方法必须在UI线程中调用\(Android5.1以前\)。
4. 不能手动调用`onPreExecute()`, `onPostExecute`, `doInBackground`, `onProgressUpdate`四个方法。
5. 一个任务实例只能执行一次。\(第二次执行会被抛出异常\)

### 变量赋值

如果一个变量是AsyncTask子类的成员变量，那么：

1. 在构造函数中赋值或者在`onPreExecute`方法中赋值，可以在所有的回调中获取该值；
2. 在`doInBackground`方法中赋值，可以在`onProgressUpdate`，`onPostExecute`，`onCancel`中获取该值。

### 执行顺序

`AsyncTask`的执行顺序随系统版本有过巨大的改变。

1. Android1.5时，它是在一个后台线程中顺序执行的，由调用顺序决定了任务的执行顺序；
2. Android1.6 - Android2.3.2它是在一个线程池当中并发执行的
3. Android3.0- ～ 它又是在一个后台线程中顺序执行。但是，如果我们想让AsyncTask并发的执行，我们可以使用`executeOnExecutor`方法，为它指定一个`Executor`对象，控制它的执行顺序。

> 可以使用以下两个Executor
>
> public static ExecutorService exec = Executors.newFixedThreadPool\(10\);
>
> AsyncTask.THREAD\_POOL\_EXECUTOR
>
> task.executeOnExecutor\(targetExecutor, params\);

## AsyncTask曾经的缺陷

AsyncTask在并发执行多个任务时发生异常。其实还是存在的，在3.0以前的系统中还是会以支持多线程并发的方式执行，支持并发数也是我们上面所计算的128，阻塞队列可以存放10个；也就是同时执行138个任务是没有问题的；而超过138会马上出现[Java](http://lib.csdn.net/base/javase).util.concurrent.RejectedExecutionException；

## 源码浅析

在进行源码解释之前先来普及一下Callable和FutureTask

### 什么是Callable和FutureTask？

Callable是类似于Runnable的接口，实现Callable和Runnable的类都是可被其他线程执行的任务。

Runnable和Callable的区别是：

\(1\)Callable规定的方法是call\(\),Runnable规定的方法是run\(\)。  
\(2\)Callable的任务执行后可返回值，而Runnable的任务是不能返回值的。  
\(3\)call方法可以抛出异常，run方法不可以。  
\(4\)运行Callable任务可以拿到一个Future对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过Future对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果。

FutureTask实际上是一个任务的操作类，它并不启动新线程，只是在自己所在线程上操作，任务的具体实现是构造FutureTask时提供的。

FutureTask执行Callable任务（类似Thread执行runnable任务），执行完会调用done方法。

我们直接从AsyncTask的execute\(Params... params\)方法说起，在这个方法里会调用executeOnExecutor\(\)方法，在这个方法中会先检查当前任务的状态是否是Pending，如果不是Pending状态，就会抛出异常，这也就意味着，一个任务不能被多次执行，如果是Pending，就将当前任务的状态变为Running状态，然后在onPreExecute\(\)方法中执行一些任务的初始化操作，之后将执行时传入的参数赋值给Callable的对象mWorker，这里的Callable对象和FutureTask对象是在创建AsyncTask时初始化的，然后调用默认执行器的execute\(\)方法，执行器在执行任务时，会先将任务加入到任务队列的队尾，然后判断mActive是否为空，第一次运行时是null，接下来就会执行scheduleNext\(\)方法，从任务队列的头部取出一个事件在线程池中执行并且将该任务在消息队列中删除，直到消息队列为空时，执行结束，在执行时，会先调用doInBackground\(\)方法，并将任务结果，通过postResult\(\)方法，借助Handler，将结果发送到UI线程中，然后执行任务的finish\(\)方法，在finish方法中根据任务是否已经被取消，决定回调onCancelled还是onPostExecute\(\)，执行完回调方法后，将该任务的状态置为Finished。

