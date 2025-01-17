# Coco format

In coco, we use `file_name` and `zip_file` to construct the file_path in `ImageDataManifest` mentioned in `README.md`. If `zip_file` is present, it means that the image is zipped into a zip file for storage & access, and the path within the zip is `file_name`. If `zip_file` is not present, the image path would just be `file_name`.

## Image classification (multiclass and multilabel)

Here is one example of the train.json, val.json, or test.json in the `DatasetInfo` above. Note that the `"id"` for `images`, `annotations` and `categories` should be consecutive integers, **starting from 1**. Note that our lib might work with id starting from 0, but many tools like [CVAT](https://github.com/openvinotoolkit/cvat/issues/2085) and official [COCOAPI](https://github.com/cocodataset/cocoapi/issues/507) will fail.

```json
{
  "images": [{"id": 1, "width": 224.0, "height": 224.0, "file_name": "train_images/siberian-kitten.jpg", "zip_file": "train_images.zip"},
              {"id": 2, "width": 224.0, "height": 224.0, "file_name": "train_images/kitten 3.jpg", "zip_file": "train_images.zip"}],
              //  file_name is the image path, which supports three formats as described in previous section.
  "annotations": [
      {"id": 1, "category_id": 1, "image_id": 1},
      {"id": 2, "category_id": 1, "image_id": 2},
      {"id": 3, "category_id": 2, "image_id": 2}
  ],
  "categories": [{"id": 1, "name": "cat"}, {"id": 2, "name": "dog"}]
}
```

## Object detection

```json
{
  "images": [{"id": 1, "width": 224.0, "height": 224.0, "file_name": "train_images/siberian-kitten.jpg", "zip_file": "train_images.zip"},
              {"id": 2, "width": 224.0, "height": 224.0, "file_name": "train_images/kitten 3.jpg", "zip_file": "train_images.zip"}],
  "annotations": [
      {"id": 1, "category_id": 1, "image_id": 1, "bbox": [10, 10, 100, 100]},
      {"id": 2, "category_id": 1, "image_id": 2, "bbox": [100, 100, 200, 200]},
      {"id": 3, "category_id": 2, "image_id": 2, "bbox": [20, 20, 200, 200]}
  ],
  "categories": [{"id": 1, "name": "cat"}, {"id": 2, "name": "dog"}]
}
```

bbox format should be **absolute** pixel position following either `ltwh: [left, top, width, height]` or `ltrb: [left, top, right, bottom]`. `ltwh` is the default format. To work with `ltrb`, please specify `bbox_format` to be `ltrb` in coco json file.

Note that

- Note that `ltrb` used to be default. If your coco annotations were prepared to work with this repo before version 0.1.2. Please add `"bbox_format": "ltrb"` to your coco file.
- Regardless of what format bboxes are stored in Coco file, when annotations are transformed into `ImageDataManifest`, the bbox will be unified into `ltrb: [left, top, right, bottom]`.

## Image caption

Here is one example of the json file for image caption task.

```json
{
  "images": [{"id": 1, "file_name": "train_images/honda.jpg", "zip_file": "train_images.zip"},
              {"id": 2, "file_name": "train_images/kitchen.jpg", "zip_file": "train_images.zip"}],
  "annotations": [
      {"id": 1, "image_id": 1, "caption": "A black Honda motorcycle parked in front of a garage."},
      {"id": 2, "image_id": 1, "caption": "A Honda motorcycle parked in a grass driveway."},
      {"id": 3, "image_id": 2, "caption": "A black Honda motorcycle with a dark burgundy seat."},
  ],
}
```

## Image text matching

Here is one example of the json file for image text matching task. `match: 1` indicates image and text match.

```json
{
  "images": [{"id": 1, "file_name": "train_images/honda.jpg", "zip_file": "train_images.zip"},
              {"id": 2, "file_name": "train_images/kitchen.jpg", "zip_file": "train_images.zip"}],
  "annotations": [
      {"id": 1, "image_id": 1, "text": "A black Honda motorcycle parked in front of a garage.", "match": 0},
      {"id": 2, "image_id": 1, "text": "A Honda motorcycle parked in a grass driveway.", "match": 0},
      {"id": 3, "image_id": 2, "text": "A black Honda motorcycle with a dark burgundy seat.", "match": 1},
  ],
}
```

## Image matting

Here is one example of the json file for image matting task. The "label" in the "annotations" can be one of the following formats: 

- a local path to the label file
- a local path in a non-compressed zip file (`c:\foo.zip@bar.png`)
- a url to the label file

Specifically, **only** image files are supported for the label files. The ground turth image should be one channel image (i.e. `PIL.Image` mode "L", instead of "RGB") that has the same width and height with the image file. Refer to the images in [tests/image_matting_test_data.zip](tests/image_matting_test_data.zip) as an example.

```json
{
    "images": [{"id": 1, "file_name": "train_images/image/test_1.jpg", "zip_file": "train_images.zip"},
                {"id": 2, "file_name": "train_images/image/test_2.jpg", "zip_file": "train_images.zip"}],
    "annotations": [
        {"id": 1, "image_id": 1, "label": "image_matting_label/mask/test_1.png", "zip_file": "image_matting_label.zip"},
        {"id": 2, "image_id": 2, "label": "image_matting_label/mask/test_2.png", "zip_file": "image_matting_label.zip"},
    ]
}
```

## Image regression

Here is one example of the json file for the image regression task, where the "target" in the "annotations" field is a real-valued number (e.g. a score, an age, etc.). Note that each image should only have one regression target (i.e. there should be exactly one annotation for each image).

```json
{
    "images": [{"id": 1, "width": 224.0, "height": 224.0, "file_name": "train_images/image_1.jpg", "zip_file": "train_images.zip"},
              {"id": 2, "width": 224.0, "height": 224.0, "file_name": "train_images/image_2.jpg", "zip_file": "train_images.zip"}],
    "annotations": [
        {"id": 1, "image_id": 1, "target": 102.0},
        {"id": 2, "image_id": 2, "target": 28.5}
    ]
}
```

## Image retrieval

This task represents data of images retrieved by text queries.

```json
{
    "images": [
        {"id": 1, "zip_file": "test1.zip", "file_name": "test/0/image_1.jpg"},
        {"id": 2, "zip_file": "test2.zip", "file_name": "test/1/image_2.jpg"}
    ],
    "annotations": [
        {"image_id": 1, "id": 1, "query": "Men eating a banana."},
        {"image_id": 2, "id": 2, "query": "An apple on the desk."}
    ]
}
```

The retrieved images might come with additional classification data associated with images in the annotation field mixed up with query annotation. This might change in future, as it can be achieved by using multitask dataset concept with one solely for image retrieval, while the other one solely for classification.


```json
{
    "images": [
        {"id": 1, "zip_file": "test1.zip", "file_name": "test/0/image_1.jpg"},
        {"id": 2, "zip_file": "test2.zip", "file_name": "test/1/image_2.jpg"}
    ],
    "categories": [
        {"id": 1, "name": "banana", "supercategory": "fruit"},
        {"id": 2, "name": "apple", "supercategory": "fruit"}
    ],
    "annotations": [
        {"image_id": 1, "id": 1, "category_id": 1, "query": "Men eating a banana."},
        {"image_id": 2, "id": 2, "category_id": 2, "query": "An apple on the desk."}
    ]
}
```

## MultiTask dataset

Multitask dataset represents the kind of dataset, where a single set of images possesses multiple sets of annotations for different tasks of single/mutiple tasks mentioned above.

For example, a set of people images can have different attributes: gender/classification {make, female, other}, height/regression: {0-300cm}, person location/detection: {x, y, w, h}, etc.

To represent this kind of dataset, it is simple: create one independent coco file for each task:

```
people_dataset/
    train_images/
        ...
    test_images/
        ...

    train_images.zip
    test_images.zip
    
    train_coco_gender.json
    test_coco_gender.json
    train_coco_height.json
    test_coco_height.json
    train_coco_location.json
    test_coco_location.json
```
```