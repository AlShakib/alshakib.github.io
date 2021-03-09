+++
toc = true
date = 2020-06-14T21:37:00+06:00
title = "A Story of Creating Drawable From Text"
description = "This is a short story about how and why I created the Android DText Library. DText (for curious mind, it is DrawableText) is a minimal Android Library that creates beautiful drawable from a string. You can use it from anywhere in your project. It is written in Java. The source code is published under GPLv3."
featured_image = "/images/story_of_android_dtext_library/dtext_logo.png"
tags = ["drawable", "dtext", "library"]
categories = ["Android"]
keywords = ["android", "library", "dtext", "drawable", "minimal", "story", "drawable", "open source"]

+++

## Introduction
Today I’m going to release the initial version of Android DText Library. It converts string text to a drawable image. You can find the GitHub repository [here](https://github.com/AlShakib/AndroidDTextLibrary).

I created DText as a part of my other personal (Yet unfinished) project. What I needed was to create an icon (or you can say avatar) with the first character of the title. Like [Contacts](https://play.google.com/store/apps/details?id=com.google.android.contacts) app from Google. As I’m a lazy human being, I was looking for a simple solution. After a few searches on Google, I found a library that does create drawables from text. [Here]( https://github.com/amulyakhare/TextDrawable) is that library  in case you are interested.

But the problem was that it was not maintained for a long time. The maintainer seems to be inactive and does not accept pull requests. Also It does not let me customize as much as I need. So, I thought why not create a new library for my present and future use.

In this article I'm going to explain how I created the DText class that converts strings to drawable.

## Initial Thoughts
What I need is a class/library that takes a string as input and gives me a nice drawable image as output. The drawable image can be a round shaped, a rounded-rectangle shape or a rectangle shape. Initially I can work with these three basic shapes, but I may implement more shapes in the future as well. Also, I need full control over the font or typeface that is going to be drawn on the canvas. I need regular, bold, italic and bold italic font weight. I may implement light font weight in the future as well.

## Getting Started

Thinking of a solution, I found out about the `ShapeDrawable` class. Android Developer Reference says,

> A Drawable object that draws primitive shapes. A ShapeDrawable takes a Shape object and manages its presence on the screen. If no Shape is given, then the ShapeDrawable will default to a RectShape. [more](https://developer.android.com/reference/android/graphics/drawable/ShapeDrawable)

In other words, it says *With ShapeDrawable, we can draw shapes as a drawable.* So, let's create a `DText` class extending the `ShapeDrawable` class and override a few methods.

```java
public class DText extends ShapeDrawable {

    @Override
    public void draw(Canvas canvas) {
        super.draw(canvas);
        // Here we will draw the actual canvas.
    }

    @Override
    public void setAlpha(int alpha) {
        // Here we will set alpha to our paint class instance.
    }

    @Override
    public void setColorFilter(ColorFilter cf) {
        // Here we will set a color filter to our paint class instance.
    }

    @Override
    public int getOpacity() {
        // Here we will return PixelFormat.TRANSLUCENT as opacity.
        // So that it can be partially transparent and should have
        // alpha blending applied.
        return PixelFormat.TRANSLUCENT;
    }

    @Override
    public int getIntrinsicWidth() {
        // Here we will return our canvas width.
        return 0;
    }

    @Override
    public int getIntrinsicHeight() {
        // Here we will return our canvas height.
        return 0;
    }
}
```

Now, we have a class called `DText` that extends `ShapeDrawable` class. We also override a few methods that we are going to use later on.

After that, we need a builder class to initialize variables and configurations. So, let's create a `Builder` class inside of the `DText` class.

```java
public class DText extends ShapeDrawable {

    private DText() {
        // Here we will initialize variables based on builder configurations.
        // We made this constructor private, because we do not want an
        // DText class instance directly.
        // Which will be useless without a proper drawable.
    }

	...

    public static class Builder {

        public Builder() {
            // Here we will initialize default variables.
        }
    }
}
```

We have builder class. Now it’s time to create some variables for configuration.

```java
public static class Builder {

    // We need a shape to draw
    private Shape shape;

    // Original text that is going to be drawn
    private String text;

    // We need a typeface for fonts
    private Typeface typeface;

    // Text and background color on the canvas
    private int textColor;
    private int backgroundColor;

    // Text size on the canvas
    private float textSize;

    // Width and height of the canvas
    private float width;
    private float height;

    public Builder() {
        // Let's initialize defaults
        text = "";
        textSize = -1;
        width = -1;
        height = -1;
        backgroundColor = Color.GRAY;
        textColor = Color.WHITE;
        shape = new RectShape(); // We will draw a rectangle shaped drawable
        typeface = Typeface.DEFAULT;
    }
    
    // We need setter methods for those variables
    public Builder setText(String text) {
        this.text = text;
        return this;
    }

    public Builder setHeight(float height) {
        this.height = height;
        return this;
    }

    public Builder setWidth(float width) {
        this.width = width;
        return this;
    }

    public Builder setTypeface(Typeface typeface) {
        this.typeface = typeface;
        return this;
    }

    public Builder setTextSize(float textSize) {
        this.textSize = textSize;
        return this;
    }

    public Builder setTextColor(int color) {
        this.textColor = color;
        return this;
    }

    public Builder setTextColor(String color) {
        this.textColor = Color.parseColor(color);
        return this;
    }

    public Builder setBackgroundColor(int color) {
        this.backgroundColor = color;
        return this;
    }

    public Builder setBackgroundColor(String color) {
        this.backgroundColor = Color.parseColor(color);
        return this;
    }

    public Builder boldText() {
        typeface = Typeface.create(typeface, Typeface.BOLD);
        return this;
    }

    public Builder italicText() {
        typeface = Typeface.create(typeface, Typeface.ITALIC);
        return this;
    }

    public Builder boldItalicText() {
        typeface = Typeface.create(typeface, Typeface.BOLD_ITALIC);
        return this;
    }

    public Builder drawAsRectangle() {
        this.shape = new RectShape();
        return this;
    }

    public Builder drawAsRound() {
        shape = new OvalShape();
        return this;
    }

    public DText build() {
        return new DText(this);
    }
}
```

Up to this, we have few customization variables in our hand. Now it’s time to actually draw on the canvas…

```java
public class DText extends ShapeDrawable {

    private final Builder builder;
    private final Paint textPaint;

    private DText(Builder builder) {
        super(builder.shape);
        this.builder = builder;

        // Initialize paint class for text
        textPaint = new Paint();
        textPaint.setAntiAlias(true);
        textPaint.setStyle(Paint.Style.FILL);
        textPaint.setTextAlign(Paint.Align.CENTER);
        textPaint.setColor(builder.textColor);
        textPaint.setTypeface(builder.typeface);

        // Initialize paint class for background
        Paint paint = getPaint();
        paint.setColor(builder.backgroundColor);
    }

    @Override
    public void draw(Canvas canvas) {
        super.draw(canvas);

        // We need drawable bounds rect
        Rect bounds = getBounds();

        // We need to save current canvas matrix
        // So that we can restore it later
        int savedCanvasCount = canvas.save();

        // Now translate our canvas from bounds rect
        canvas.translate(bounds.left, bounds.top);

        // If builder class does not provide a valid width and height,
        // use bounds rect's width and height as canvas's width and height.
        // We want to fill the whole drawable by default.
        float canvasWidth = builder.width < 0 ? bounds.width() : builder.width;
        float canvasHeight = builder.height < 0 ? bounds.height() : builder.height;

        // If builder class does not provide a valid text size,
        // calculate text size from canvas width and height.
        // Use half of the lowest dimension as text size by default.
        // It can be width or height.
        float textSize = builder.textSize < 0 ?
                (Math.min(canvasWidth, canvasHeight) / 2) : builder.textSize;
        textPaint.setTextSize(textSize);

        // Now it's time to draw the text on the canvas.
        canvas.drawText(builder.text, canvasWidth / 2, canvasHeight / 2 -
                ((textPaint.descent() + textPaint.ascent()) / 2), textPaint);

        // Restore previous matrix to the canvas
        canvas.restoreToCount(savedCanvasCount);
    }

    @Override
    public void setAlpha(int alpha) {
        // Set alpha to our paint class instance.
        textPaint.setAlpha(alpha);
    }

    @Override
    public void setColorFilter(ColorFilter cf) {
        // Set color filter to our paint class instance.
        textPaint.setColorFilter(cf);
    }

    @Override
    public int getOpacity() {
        // Here we will return PixelFormat.TRANSLUCENT as opacity.
        // So that it can be partially transparent and should have
        // alpha blending applied.
        return PixelFormat.TRANSLUCENT;
    }

    @Override
    public int getIntrinsicWidth() {
        // Return our canvas width.
        return (int) builder.width;
    }

    @Override
    public int getIntrinsicHeight() {
        // Return our canvas height.
        return (int) builder.height;
    }

    public static class Builder {

        // We need a shape to draw
        private Shape shape;

        // Original text that is going to be drawn
        private String text;

        // We need a typeface for fonts
        private Typeface typeface;

        // Text and background color on the canvas
        private int textColor;
        private int backgroundColor;

        // Text size on the canvas
        private float textSize;

        // Width and height of the canvas
        private float width;
        private float height;

        public Builder() {
            // Let's initialize defaults
            text = "";
            textSize = -1;
            width = -1;
            height = -1;
            backgroundColor = Color.GRAY;
            textColor = Color.WHITE;
            shape = new RectShape(); // We will draw a rectangle shaped drawable
            typeface = Typeface.DEFAULT;
        }

        public Builder setText(String text) {
            this.text = text;
            return this;
        }

        public Builder setHeight(float height) {
            this.height = height;
            return this;
        }

        public Builder setWidth(float width) {
            this.width = width;
            return this;
        }

        public Builder setTypeface(Typeface typeface) {
            this.typeface = typeface;
            return this;
        }

        public Builder setTextSize(float textSize) {
            this.textSize = textSize;
            return this;
        }

        public Builder setTextColor(int color) {
            this.textColor = color;
            return this;
        }

        public Builder setTextColor(String color) {
            this.textColor = Color.parseColor(color);
            return this;
        }

        public Builder setBackgroundColor(int color) {
            this.backgroundColor = color;
            return this;
        }

        public Builder setBackgroundColor(String color) {
            this.backgroundColor = Color.parseColor(color);
            return this;
        }

        public Builder boldText() {
            typeface = Typeface.create(typeface, Typeface.BOLD);
            return this;
        }

        public Builder italicText() {
            typeface = Typeface.create(typeface, Typeface.ITALIC);
            return this;
        }

        public Builder boldItalicText() {
            typeface = Typeface.create(typeface, Typeface.BOLD_ITALIC);
            return this;
        }

        public Builder drawAsRectangle() {
            this.shape = new RectShape();
            return this;
        }

        public Builder drawAsRound() {
            shape = new OvalShape();
            return this;
        }

        public DText build() {
            return new DText(this);
        }
    }
}
```

## Testing

This is the full `DText` class up to now. Let’s test it in our activity class. We have an `ImageView` as `demoImageView`. Now, let's create a drawable text using `DText` class and set it as an image drawable to the `ImageView`.

```Java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    demoImageView = findViewById(R.id.demo_image_view);
    Drawable drawableText = new DText.Builder()
        .setText("First")
        .setTextColor(Color.RED)
        .setBackgroundColor(Color.BLUE)
        .build();
    demoImageView.setImageDrawable(drawableText);
}
```

And voila... We have a drawable which is created from a text.

{{< figure src="/images/story_of_android_dtext_library/android_dtext_library_1.jpg" caption="First drawable from a text" width="250" >}}

## Adding Customization

Up to this, we have a fully working DText class that creates drawables from text. Now we can add customizations like,

```java
// Create an instance of Builder class.
DText.Builder builder = new DText.Builder();

// Set text that we are going to draw.
builder.setText("Android DText Library");

// Draw only the first character.
// If the text is "android",
// then the builder will draw "a" on the canvas.
builder.firstCharOnly();

// Draw only the alphanumeric character.
// If the text is "<Unknown>",
// then the builder will draw first alphanumeric character
// from the text. In this case, it is "U".
// NOTE: alphaNumOnly() will not work without firstCharOnly().
builder.alphaNumOnly();

// Draw only the first digit from the text.
// If the text is "You have 5 notifications",
// then the builder will draw "5" on the canvas.
// NOTE: digitOnly() will not work without firstCharOnly().
builder.digitOnly();

// Use random background color from a nice preset background color list.
builder.randomBackgroundColor();

// You can pass your own color list to the builder as well.
List<String> colorList = new ArrayList<>();
colorList.add("#9C27B0");
colorList.add("#EF6C00");
builder.setRandomColorList(colorList);

// You can set a background color as well.
// By default, the background color is gray.
builder.setBackgroundColor(Color.BLUE);
// or
builder.setBackgroundColor("#0000FF");

// You can set text color with integer or string (HEX) value.
// By default, the text color is white.
builder.setTextColor(Color.RED);
// or
builder.setTextColor("#FF0000");

// Set height and width of the canvas.
builder.setHeight(150);
builder.setWidth(150);

// Set text size.
builder.setTextSize(24);

// By default, DText uses pixels to calculate height, width and text size.
// But you can use DP for height/width and SP for text size as well
// by passing a context to the builder.
builder.useSpAndDp(context);

// Use bold text.
builder.boldText();

// Use italic text.
builder.italicText();

// Use bold italic text.
builder.boldItalicText();

// By default, DText uses Typeface.DEFAULT.
// But you can pass your own typeface as well.
builder.setTypeface(Typeface.create(Typeface.MONOSPACE, Typeface.NORMAL));

// Transform to upper case letter.
builder.toUpperCase();

// Draw as a round on the canvas.
builder.drawAsRound();

// Draw as a rectangle on the canvas.
builder.drawAsRectangle();

// Draw as a rectangle with border radius on the canvas.
builder.drawAsRectangle(16);
```

After creating our final library we can use it like these..

{{< figure src="/images/story_of_android_dtext_library/android_dtext_library_2.jpg" caption="First letter avatar with round shape" width="250" >}}

## Final Words

Thanks for reading this article. If you have any question or confusion regarding the tutorial, feel free to ask your questions on [Telegram](https://t.me/AlShakib) or [Twitter](https://twitter.com/_alshakib). You can send me an email as well.
