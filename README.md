# CNN-image-recognition
Certainly! Here's an example of how you can implement image recognition using a Convolutional Neural Network (CNN) in Java. In this example, we'll use the Deep Java Library (DJL), an open-source library that provides support for deep learning in Java.

To get started, you'll need to include the DJL dependency in your project. You can find the latest version on the DJL GitHub repository.

```xml
<dependency>
    <groupId>ai.djl</groupId>
    <artifactId>api</artifactId>
    <version>0.16.0</version>
</dependency>
<dependency>
    <groupId>ai.djl.tensorflow</groupId>
    <artifactId>tensorflow-engine</artifactId>
    <version>0.16.0</version>
</dependency>
```

Now, let's create a simple image recognition program using a pre-trained model (ResNet) from DJL:

```java
import ai.djl.Application;
import ai.djl.Model;
import ai.djl.ModelException;
import ai.djl.ModelZoo;
import ai.djl.modality.Classifications;
import ai.djl.modality.Classifications.Classification;
import ai.djl.modality.Classifications.Classifier;
import ai.djl.modality.Image;
import ai.djl.modality.Image.ImageBuilder;
import ai.djl.modality.ImageVisualization;
import ai.djl.modality.cv.ImageFactory;
import ai.djl.modality.cv.ImageTransform;
import ai.djl.modality.cv.ImageTransforms;
import ai.djl.modality.cv.ImageTransforms.Resize;
import ai.djl.modality.cv.ImageTransforms.ToTensor;
import ai.djl.modality.cv.ImageTransforms.Normalize;
import ai.djl.translate.TranslateException;
import ai.djl.translate.Translator;
import ai.djl.util.Utils;
import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.List;

public class CNNImageRecognitionExample {
    public static void main(String[] args) throws ModelException, IOException, TranslateException {
        // Load a pre-trained ResNet model
        Model model = ModelZoo.loadModel(Application.CV.IMAGE_CLASSIFICATION, "resnet");

        // Create a Translator to preprocess input images
        Translator<Image, Classifications> translator = new Translator<>() {
            @Override
            public Image processInput(TranslatorContext ctx, Image input) {
                return input.toTensor();
            }

            @Override
            public Classifications processOutput(TranslatorContext ctx, ai.djl.ndarray.NDList list) {
                Classifier<Image, Classifications> classifier = new Classifier<>(list.singletonOrThrow());
                return classifier;
            }

            @Override
            public Batchifier getBatchifier() {
                return null;
            }
        };

        // Load and preprocess an image
        ImageBuilder builder = ImageFactory.getInstance().fromUrl("https://djl-ai.s3.amazonaws.com/resources/images/kitten.jpg");
        builder.transform(new Resize(224, 224)).transform(new ToTensor());

        // Predict
        Classifications classifications = model.predict(builder.build(), translator).get(0);

        // Print results
        List<Classification> items = classifications.items();
        for (int i = 0; i < 3; i++) { // Display top 3 results
            Classification item = items.get(i);
            System.out.println(String.format("Label: %s, Probability: %.5f", item.getClassName(), item.getProbability()));
        }

        // Display the input image
        ImageVisualization.drawBoundingBoxes(builder.build(), classifications);
    }
}
```

This example uses a pre-trained ResNet model to classify an image using DJL. It loads the model, preprocesses the input image, and then prints the top predicted labels and their probabilities. Additionally, it visualizes the image with bounding boxes around the recognized objects.

Please note that you need to have an active internet connection to download the pre-trained model for the first time. Also, make sure to import the required DJL packages and add the necessary dependencies as mentioned earlier.
