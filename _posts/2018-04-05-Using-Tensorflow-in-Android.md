---
layout: post
title: Using Tensorflow in Android
description : In this post I will be discussing about how to use the saved machine learning model in android so as to apply machine learning in mobile applications.
tags: [Tensorflow, Android]
author : Ayush Agarwal
time : 10 minutes read
---

As we all know Google has open-sourced a library called TensorFlow that can be used in Android for implementing Machine Learning.

>  TensorFlow is an open-source software library for Machine Intelligence provided by Google.

I searched the internet a lot but did not find a simple way or a simple example to build TensorFlow for Android. After going through many resources, I was able to build it. Then, I decided to write on it so that it would not take time for others.

**Credit :** The classifier example has been taken from `Google TensorFlow example`.

This article is for those who are already familiar with machine learning and know how to the build model for machine learning(for this example I will be using a pre-trained model). Sooner, I will be writing a series of articles on machine learning so that everybody can learn how to build the model for machine learning. In this post I won't be discussing how to make the tensorflow classifier but using the saved model in android so as to apply machine learning in mobile applications.

You can clone my app using git using the command in git bash:

```groovy
git clone https://github.com/aagarwal1012/Tensorflow-In-Android.git
```

## Let's Get Started

First you need to add some line after jcenter() in your project build.gradle

```groovy
maven {
            url 'https://maven.google.com/'
            name 'Google'
        }
```

First you need to add some dependencies in your app build.gradle

```groovy
    //tensorflow library
    compile 'org.tensorflow:tensorflow-android:1.2.0'

    //used for camera activity
    compile 'com.wonderkiln:camerakit:0.13.1'
```

Next you have to add two files in "assets" folder in app main. First one is [tensorflow_inception_graph.pb](https://drive.google.com/file/d/12WMNdO6fNPr1w8ouBAhpOZyFPOnZGC5W/view?usp=sharing) which is the saved tensorflow graph and second one is [imagenet_comp_graph_label_strings.txt](https://drive.google.com/file/d/1bpsBNtekJvHZ5ac0wrd1dYwppZGz7xHc/view?usp=sharing), it contains all the labels.

## Now Its time to code

Add some code in the **activity_main.xml**

<script src="https://gist.github.com/aagarwal1012/b8d2e08a36c50f69af7c11b17065ae2a.js"></script>

Now create a interface named **Classifier.java.** In this, we will add a class Recognition which contains the variable id, title, confidence and location that are the feature of the classified object. Also add some getters and setters to the class Recognition.

Now add some abstract method which we later gonna define in our TensorFlowImageClassifier class.

After all this, now our code should look like this

```java
package com.ayush.tensorflowclassifier;

import android.graphics.Bitmap;
import android.graphics.RectF;

import java.util.List;

/**
 * Generic interface for interacting with different recognition engines.
 */
public interface Classifier {
    /**
     * An immutable result returned by a Classifier describing what was recognized.
     */
    public class Recognition {
        /**
         * A unique identifier for what has been recognized. Specific to the class, not the instance of
         * the object.
         */
        private final String id;

        /**
         * Display name for the recognition.
         */
        private final String title;

        /**
         * A sortable score for how good the recognition is relative to others. Higher should be better.
         */
        private final Float confidence;

        /**
         * Optional location within the source image for the location of the recognized object.
         */
        private RectF location;

        public Recognition(
                final String id, final String title, final Float confidence, final RectF location) {
            this.id = id;
            this.title = title;
            this.confidence = confidence;
            this.location = location;
        }

        public String getId() {
            return id;
        }

        public String getTitle() {
            return title;
        }

        public Float getConfidence() {
            return confidence;
        }

        public RectF getLocation() {
            return new RectF(location);
        }

        public void setLocation(RectF location) {
            this.location = location;
        }

        @Override
        public String toString() {
            String resultString = "";
            if (id != null) {
                resultString += "[" + id + "] ";
            }

            if (title != null) {
                resultString += title + " ";
            }

            if (confidence != null) {
                resultString += String.format("(%.1f%%) ", confidence * 100.0f);
            }

            if (location != null) {
                resultString += location + " ";
            }

            return resultString.trim();
        }
    }

    List recognizeImage(Bitmap bitmap);

    void enableStatLogging(final boolean debug);

    String getStatString();

    void close();
}
```

Now, create a class named **TensorFlowImageClassifier.java** which implements our previously build class Classifier.

In this I had comment each line so it can be easily understandable. Code follows:

```java
package com.ayush.tensorflowclassifier;

import android.content.res.AssetManager;
import android.graphics.Bitmap;
import android.support.v4.os.TraceCompat;
import android.util.Log;

import org.tensorflow.contrib.android.TensorFlowInferenceInterface;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.PriorityQueue;
import java.util.Vector;

/**
 * A classifier specialized to label images using TensorFlow.
 */
public class TensorFlowImageClassifier implements Classifier {

    private static final String TAG = "ImageClassifier";

    // Only return this many results with at least this confidence.
    private static final int MAX_RESULTS = 3;
    private static final float THRESHOLD = 0.1f;

    // Config values.
    private String inputName;
    private String outputName;
    private int inputSize;
    private int imageMean;
    private float imageStd;

    // Pre-allocated buffers.
    private Vector labels = new Vector();  //store the lables present in imagenet_comp_graph_label_strings.txt
    private int[] intValues;
    private float[] floatValues;
    private float[] outputs;
    private String[] outputNames;

    //From the tensorflow library
    private TensorFlowInferenceInterface inferenceInterface;

    private boolean runStats = false;

    private TensorFlowImageClassifier() {
    }

    /**
     * Initializes a native TensorFlow session for classifying images.
     *
     * @param assetManager  The asset manager to be used to load assets.
     * @param modelFilename The filepath of the model GraphDef protocol buffer.
     * @param labelFilename The filepath of label file for classes.
     * @param inputSize     The input size. A square image of inputSize x inputSize is assumed.
     * @param imageMean     The assumed mean of the image values.
     * @param imageStd      The assumed std of the image values.
     * @param inputName     The label of the image input node.
     * @param outputName    The label of the output node.
     * @throws IOException
     */

    public static Classifier create(
            AssetManager assetManager,
            String modelFilename,
            String labelFilename,
            int inputSize,
            int imageMean,
            float imageStd,
            String inputName,
            String outputName)
            throws IOException {
        TensorFlowImageClassifier c = new TensorFlowImageClassifier();
        c.inputName = inputName;
        c.outputName = outputName;

        // Read the label names into memory.
        String actualFilename = labelFilename.split("file:///android_asset/")[1];
        Log.i(TAG, "Reading labels from: " + actualFilename);
        BufferedReader br = null;
        br = new BufferedReader(new InputStreamReader(assetManager.open(actualFilename)));
        String line;
        while ((line = br.readLine()) != null) {
            c.labels.add(line);
        }
        br.close();

        c.inferenceInterface = new TensorFlowInferenceInterface(assetManager, modelFilename);
        // The shape of the output is [N, NUM_CLASSES], where N is the batch size.
        int numClasses = (int)c.inferenceInterface.graph().operation(outputName).output(0).shape().size(1);
        Log.i(TAG, "Read " + c.labels.size() + " labels, output layer size is " + numClasses);

        // Ideally, inputSize could have been retrieved from the shape of the input operation.  Alas,
        // the placeholder node for input in the graphdef typically used does not specify a shape, so it
        // must be passed in as a parameter.
        c.inputSize = inputSize;
        c.imageMean = imageMean;
        c.imageStd = imageStd;

        // Pre-allocate buffers.
        c.outputNames = new String[]{outputName};
        c.intValues = new int[inputSize * inputSize];
        c.floatValues = new float[inputSize * inputSize * 3];
        c.outputs = new float[numClasses];

        return c;
    }

    //overriding methods from Classifier interface

    @Override
    public List recognizeImage(final Bitmap bitmap) {

        // Log this method so that it can be analyzed with systrace.
        TraceCompat.beginSection("recognizeImage");

        TraceCompat.beginSection("preprocessBitmap");
        // Preprocess the image data from 0-255 int to normalized float based
        // on the provided parameters.
        bitmap.getPixels(intValues, 0, bitmap.getWidth(), 0, 0, bitmap.getWidth(), bitmap.getHeight()); //getting the pixel value from bitmap and storing in array intValues
        for (int i = 0; i < intValues.length; ++i) {                                                               //pre-processing intValues
            final int val = intValues[i];
            floatValues[i * 3 + 0] = (((val >> 16) & 0xFF) - imageMean) / imageStd;
            floatValues[i * 3 + 1] = (((val >> 8) & 0xFF) - imageMean) / imageStd;
            floatValues[i * 3 + 2] = ((val & 0xFF) - imageMean) / imageStd;
        }
        TraceCompat.endSection();

        // Copy the input data into TensorFlow.
        TraceCompat.beginSection("feed");
        inferenceInterface.feed(
                inputName, floatValues, new long[]{1, inputSize, inputSize, 3});
        TraceCompat.endSection();

        // Run the inference call.
        TraceCompat.beginSection("run");
        inferenceInterface.run(outputNames, runStats);
        TraceCompat.endSection();

        // Copy the output Tensor back into the outputs array.
        TraceCompat.beginSection("fetch");
        inferenceInterface.fetch(outputName, outputs);
        TraceCompat.endSection();

        // Find the best classifications using the priority queue data-structure
        //making the priority queue

        PriorityQueue pq =
                new PriorityQueue(
                        3,
                        new Comparator() {
                            @Override
                            public int compare(Recognition lhs, Recognition rhs) {
                                // Intentionally reversed to put high confidence at the head of the queue.
                                return Float.compare(rhs.getConfidence(), lhs.getConfidence());
                            }
                        });

        //output only best 3
        for (int i = 0; i < outputs.length; ++i) {
            if (outputs[i] > THRESHOLD) {
                pq.add(
                        new Recognition(
                                "" + i, labels.size() > i ? labels.get(i) : "unknown", outputs[i], null));
            }
        }

        final ArrayList recognitions = new ArrayList();
        int recognitionsSize = Math.min(pq.size(), MAX_RESULTS);
        //getting values from pq and storing in ArrayList
        for (int i = 0; i < recognitionsSize; ++i) {
            recognitions.add(pq.poll());
        }

        TraceCompat.endSection(); // "recognizeImage"

        return recognitions;
    }

    @Override
    public void enableStatLogging(boolean debug) {
        runStats = debug;
    }

    @Override
    public String getStatString() {
        return inferenceInterface.getStatString();
    }

    @Override
    public void close() {
        inferenceInterface.close();
    }
}
```

So finally we are going in the final part in which we will write some code in **MainActivity.java**. Code follows:

```java
package com.ayush.tensorflowclassifier;

import android.graphics.Bitmap;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.text.method.ScrollingMovementMethod;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.TextView;

import com.wonderkiln.camerakit.CameraKitError;
import com.wonderkiln.camerakit.CameraKitEvent;
import com.wonderkiln.camerakit.CameraKitEventListener;
import com.wonderkiln.camerakit.CameraKitImage;
import com.wonderkiln.camerakit.CameraKitVideo;
import com.wonderkiln.camerakit.CameraView;

import java.util.List;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

public class MainActivity extends AppCompatActivity {

    //defining constants
    private static final int INPUT_SIZE = 224;
    private static final int IMAGE_MEAN = 117;
    private static final float IMAGE_STD = 1;
    private static final String INPUT_NAME = "input";
    private static final String OUTPUT_NAME = "output";

    private static final String MODEL_FILE = "file:///android_asset/tensorflow_inception_graph.pb";
    private static final String LABEL_FILE =
            "file:///android_asset/imagenet_comp_graph_label_strings.txt";

    //creating objects
    private Classifier classifier;
    private Executor executor = Executors.newSingleThreadExecutor();
    private TextView textViewResult;
    private Button btnDetectObject, btnToggleCamera;
    private ImageView imageViewResult;
    private CameraView cameraView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //defining created objects
        cameraView = (CameraView) findViewById(R.id.cameraView);
        imageViewResult = (ImageView) findViewById(R.id.imageViewResult);
        textViewResult = (TextView) findViewById(R.id.textViewResult);
        textViewResult.setMovementMethod(new ScrollingMovementMethod());

        btnToggleCamera = (Button) findViewById(R.id.btnToggleCamera);
        btnDetectObject = (Button) findViewById(R.id.btnDetectObject);

        //cameraview
        cameraView.addCameraKitListener(new CameraKitEventListener() {
            @Override
            public void onEvent(CameraKitEvent cameraKitEvent) {

            }

            @Override
            public void onError(CameraKitError cameraKitError) {

            }

            @Override
            public void onImage(CameraKitImage cameraKitImage) {

                Bitmap bitmap = cameraKitImage.getBitmap();

                //scalling bitmap to Input_Size*Input_Size
                bitmap = Bitmap.createScaledBitmap(bitmap, INPUT_SIZE, INPUT_SIZE, false);

                imageViewResult.setImageBitmap(bitmap);

                //recognizing bitmap
                final List results = classifier.recognizeImage(bitmap);

                //displaying it in textView
                textViewResult.setText(results.toString());

            }

            @Override
            public void onVideo(CameraKitVideo cameraKitVideo) {

            }
        });

        //toggle button
        btnToggleCamera.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                cameraView.toggleFacing();
            }
        });

        //detect button
        btnDetectObject.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                cameraView.captureImage();
            }
        });

        initTensorFlowAndLoadModel();
    }

    @Override
    protected void onResume() {
        super.onResume();
        cameraView.start();
    }

    @Override
    protected void onPause() {
        cameraView.stop();
        super.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        executor.execute(new Runnable() {
            @Override
            public void run() {
                classifier.close();
            }
        });
    }

    //inttilizing model
    private void initTensorFlowAndLoadModel() {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    classifier = TensorFlowImageClassifier.create(
                            getAssets(),
                            MODEL_FILE,
                            LABEL_FILE,
                            INPUT_SIZE,
                            IMAGE_MEAN,
                            IMAGE_STD,
                            INPUT_NAME,
                            OUTPUT_NAME);
                    makeButtonVisible();
                } catch (final Exception e) {
                    throw new RuntimeException("Error initializing TensorFlow!", e);
                }
            }
        });
    }

    private void makeButtonVisible() {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                btnDetectObject.setVisibility(View.VISIBLE);
            }
        });
    }
}
```

## Testing

Finally, we finished with the coding part now it's the time to run it. So let it build and run in your Android Device.

Here are some Screenshots of the app.

<img src = "{{ site.baseurl }}/assets/img/post/post1_image1.png" width = "50%">


<img src = "{{ site.baseurl }}/assets/img/post/post1_image2.png" width = "50%"> 

If you find any difficulty in the tutorial or any problem or error in installation and validation do not hesitate to post a comment below.
