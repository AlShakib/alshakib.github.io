+++
toc = true
date = 2020-07-18T22:21:38+06:00
title = "Let Me Introduce Tasker"
description = "Tasker is intended to enable proper and easy use of the UI and background thread. It lets you perform operations in the background. When they’ve finished running, it then allows you to update views in the main event thread. In most applications, you’ll need a way to update its data. If you’re not careful, your application will do almost all of its work on the main event thread because this thread runs your event methods. If you just drop your heavy task code into the main event thread, then the main event thread will be busy with the heavy task, instead of rushing off to look for any events from the screen or other applications. If your heavy task code takes a long time to run, users will feel like the application has crashed. So, the trick is to move the heavy task code off the main event thread and run it a custom thread in the background."
featured_image = "/images/android_tasker_library/tasker_cover.png"
tags =  ["thread", "tasker", "library"]
categories = ["Android"]
keywords = ["android", "library", "tasker", "asynctask", "thread", "background thread", "java", "open source"]
slug = "let_me_introduce_tasker"

+++

## Introduction

In most applications, you’ll need a way to update its data. If you’re not careful, your application will do almost all of its work on the main event thread because this thread runs your event methods. If you just drop your heavy task code into the main event thread, then the main event thread will be busy with the heavy task, instead of rushing off to look for any events from the screen or other applications. If your heavy task code takes a long time to run, users will feel like the application has crashed. So, the trick is to move the heavy task code off the main event thread and run it a custom thread in the background.

Up to Android SDK 29, you could use `AsyncTask` to write efficient multithreaded code to keep your application speedy. But the `AsyncTask` class was deprecated in API level 30. Android reference suggests to use the standard `java.util.concurrent` utilities instead. They recommended the various APIs provided by the `java.util.concurrent` package such as `Executor`, `ThreadPoolExecutor` and `FutureTask`.

`Tasker` is intended to enable proper and easy use of the UI and background thread. It lets you perform operations in the background. When they’ve finished running, it then allows you to update views in the main event thread.

## Getting Started

To use `Tasker` in your project, follow installation instructions from [here](https://github.com/AlShakib/Tasker). The latest version is available for Android SDK 14 and higher.

### Usages
For basic usages, you'll need to create a `Tasker`  object, and pass a `Tasker.Task<Result>` object to the `executeAsync()` method.

You can create a `Tasker.Task<Result>` class by extending the `Tasker.Task<Result>` class, and implementing its `doInBackground()` method. The code in this method runs in a background thread, so it's the perfect place for you to put your code for a heavy job. The `Tasker.Task<Result>` class also has an `onPreExecute()` method that runs before `doInBackground()` and an `onPostExecute()` method that runs afterward.

`Tasker.Task` is defined by a generic parameter: Result which is the type of the task result. You can set this to Void if you're not going to use it.

```java
class MyTask extends Tasker.Task<Result> {

    @Override
    protected void onPreExecute() {
        // Code to run before executing the task.
    }

    @Override
    protected Result doInBackground() {
        // Code that you want to run in a background thread.
        return null;
    }

    @Override
    protected void onPostExecute(Result result) {
        // Code that you want to run when the task is complete.
    }
}
```

You can run a Task by calling the Tasker `executeAsync()` method and passing it a `Tasker.Task` object.

```java
Tasker tasker = new Tasker();
tasker.executeAsync(new MyTask());
```

### The `onPreExecute()` Method

This gets called before the background task begins, and it’s used to set up the task. It’s called on the main event thread, so it has access to views in the user interface. The `onPreExecute()` method takes no parameters, and has a void return type. For example,

```java
@Override
protected void onPreExecute() {

    // Show progress bar before start downloading.
    progressBar.setVisibility(View.VISIBLE);
}

```

### The `doInBackground()` Method

The `doInBackground()` method runs in the background immediately after `onPreExecute()`. You can define what type of parameters the task should receive, and what the return type should be. For example,

```java
@Override
protected Bitmap doInBackground() {

    // Download the image in the background
    // and then convert it to Bitmap.
    Bitmap result = null;
    try {
        URL ImageUrl = new URL(getResources().getString(R.string.image_url));
        HttpURLConnection conn = (HttpURLConnection) ImageUrl.openConnection();
        conn.setDoInput(true);
        conn.connect();
        InputStream inputStream = conn.getInputStream();
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inPreferredConfig = Bitmap.Config.RGB_565;
        result = BitmapFactory.decodeStream(inputStream, null, options);
    } catch (IOException e) {
        e.printStackTrace();
    }
    return result;
}
```

### The `onPostExecute()` Method

The `onPostExecute()` method is called after the background task has finished. You can use this method to present the results of the task to the user. The `onPostExecute()` method gets passed the results of the `doInBackground()` method, so it must take parameters that match the `doInBackground()` return type. For example,

```java
@Override
protected void onPostExecute(Bitmap result) {

    // When the doInBackground method is completed,
    // this method is going to be called.
    // Here we will set result to the image view.
    progressBar.setVisibility(View.GONE);
    
    if (result != null) {
        resultImageView.setImageBitmap(result);
    } else {
        resultTextView.setText(getResources().getString(R.string.result_error));
    }
}
```

## Final Words

Thanks for reading this article. If you have any question or confusion regarding the tutorial, feel free to ask your questions on [Telegram](https://t.me/AlShakib) or [Twitter](https://twitter.com/_alshakib). You can send me an email as well.
