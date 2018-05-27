![wasmface](https://github.com/noahlevenson/wasmface/blob/master/wasmface.gif)

### wasmface

Demo: http://wasmface.noahlevenson.com

Wasmface is a fast computer vision library for detecting human faces and other objects in an HTML5 canvas element.

It's a WebAssembly implementation of the [Viola-Jones framework](https://www.cs.cmu.edu/~efros/courses/LBMV07/Papers/viola-cvpr-01.pdf) for rapid object detection. It's written in C++ for the Emscripten compiler.

For JavaScript developers: You don't need to know C++ or wasm conventions to use Wasmface! Wasmface provides JavaScript wrappers that handle memory management and bindings. Wasmface cascade classifier models are formatted as JSON.

For C++ developers: The Wasmface engine is written from scratch in C++17. Optimize it for your use case!

Wasmface comes with [human-face.js](https://github.com/noahlevenson/wasmface/blob/master/models/human-face.js), a **work-in-progress** cascade classifier model trained to detect faces. Early iterations of the model have been trained on ~13,000 positive examples from [LFW-a](https://www.openu.ac.il/home/hassner/data/lfwa/) and ~10,000 negative examples generated from stock photos.

You can also create your own models using [wasmface-trainer](https://github.com/noahlevenson/wasmface/blob/master/src/cpp/wasmface-trainer.cpp), a native executable tool that implements the training phase of the framework. Viola-Jones is suitable for detecting a variety of object classes.

Features:

* Absolutely no GPU acceleration :stuck_out_tongue_winking_eye: Wasmface is an experiment in web CPU performance
* Detection merging via non-maximum suppression
* Variance normalization (pre-applied during training, post-applied during detection)
* 5 types of Haar-like features
* Optimized for HTML5 ImageData single-channel pseudograyscale (luma in 4th byte)
* Integral images, feature scaling, and everything else described in the 2001 paper

I developed Wasmface as part of my research at [Recurse Center](https://recurse.com), which focused on WebAssembly in the domain of computer vision.

#### :rocket: getting started
Here's how to get Wasmface up and running:
```html
<script src="wasmface.js"></script> // Glue code that loads the .wasm module
<script src="wasmface-wrap.min.js"></script> // JS wrappers
<script src="human-face.js"></script> // Cascade classifier model for detecting faces

<canvas id="canvas"></canvas>

<script>
	const canvas = document.getElementById("canvas");
	const ctx = canvas.getContext("2d");

	const myWasmface = new Wasmface(humanFace); // Instantiate a new Wasmface object
	const boundingBoxes = myWasmface.detect(ctx); // Detect objects in a canvas and get bounding boxes!
</script>
```

#### :smiley: using wasmface
The Wasmface JavaScript interface was designed with simplicity in mind. Instantiate a new Wasmface object by passing the constructor a JSON Wasmface cascade classifier model:

```javascript
const myWasmface = new Wasmface(humanFace);
```

##### **Methods**

##### detect(ctx, [pp, othresh, nthresh, step, delta])

Use a cascade classifier model to detect objects in a canvas element.

`ctx` The 2D canvas context

`pp` 1 applies post processing, 0 does not. Post processing uses non-maximum suppression to reduce duplicate detections around regions of interest.

`othresh` Overlap threshold for post processing - the minimum ratio of overlap required to suppress a bounding box. 

`nthresh` Neighbor threshold for post processing - the minimum number of neighbors required to suppress a bounding box.

`step` Detector scale step size.

`delta` Detector sweep delta size.

##### destroy()

Manually deallocate the heap memory associated with a cascade classifier. 

#### :boom: using wasmface-trainer
```
wasmface-trainer --b 24 --s 10000 --p /path/to/positives --n /path/to/negatives --pv /path/to/validation/positives --pn /path/to/validation/negatives
```

`--b` **Base resolution**

The base resolution at which to train your model.

`--s` **Negative set size**

The size of the negative training set. The negative training set is generated by extracting subwindows from negative training examples. After training each layer of the cascade, the algorithm will discard all images from the negative training set that are correctly classified, then add false positives to the negative training set until it achieves the specified negative set size.

`--p` **Path to positive training examples** 

A directory containing positive training images in .jpg or .ppm format. Any subdirectories will be recursively scanned for images. Images are assumed to be 1:1 aspect ratio and of arbitrary size.

`--n` **Path to negative training examples**

A directory containing negative training images in .jpg or .ppm format. Any subdirectories will be recursively scanned for images. Images are assumed to be larger than the specified base resolution and of arbitrary aspect ratio.

`--pv` **Path to positive validation examples**

A directory containing positive validation images in .jpg or .ppm format. Any subdirectories will be recursively scanned for images. Images are assumed to be 1:1 aspect ratio and of arbitrary size.

`--pn` **Path to negative validation examples**

A directory containing negatie validation images in .jpg or .ppm format. Any subdirectories will be recursively scanned for images. Images are assumed to be larger than the specified base resolution and of arbitrary aspect ratio.

#### :floppy_disk: compiling from source
**wasmface**
```
emcc wasmface.cpp cascade-classifier.cpp haar-like.cpp integral-image.cpp strong-classifier.cpp utility.cpp weak-classifier.cpp -s TOTAL_MEMORY=1024MB -s "EXTRA_EXPORTED_RUNTIME_METHODS=['ccall', 'cwrap', 'allocate']" -s WASM=1 -O3 -std=c++1z -o wasmface.js
```
**wasmface-trainer**
```
g++ wasmface-trainer.cpp utility.cpp integral-image.cpp haar-like.cpp weak-classifier.cpp strong-classifier.cpp cascade-classifier.cpp -O3 -lpthread -std=c++17 "-lstdc++fs" -o wasmface-trainer
```
#### :books: dependencies
[JSON for Modern C++](https://github.com/nlohmann/json): Used to construct JSON objects during model serialization and deserialization.

[CImg](https://github.com/dtschump/CImg): Used during training for loading and manipulating local image files. CImg depends on [ImageMagick](https://github.com/ImageMagick/ImageMagick) to decode .jpg and .ppm files.

#### :memo: todo
- [ ] Overall optimization
- [ ] Parameterize wasmface-trainer for layer count, max features and target FPR
- [ ] Sanitization of wasmface-trainer arguments
- [ ] Implement pause and resume for wasmface-trainer
- [ ] Train a vastly improved human face classifier

#### :pager: contact
[twitter.com/noahlevenson](https://twitter.com/noahlevenson)

noahlevenson [at] gmail [dot] com