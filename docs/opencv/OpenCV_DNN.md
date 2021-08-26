# OpenCV DNN
## Overview of DNN Module
* DNN (Deep Neural Network) module was initially part of opencv_contrib repo. 
* It has been moved to the master branch of opencv repo last year, giving users the ability to run inference on pre-trained deep learning models within OpenCV itself.
* One thing to note here is, **dnn module** is not meant be used for training. **It’s just for running inference on images/videos**.
* Initially only Caffe and Torch models were supported. Over the period support for different frameworks/libraries like TensorFlow is being added.
* **Support for YOLO/DarkNet has been added recently**

## Detection from the Image
```
detect_image.py
```
* Load model and related parameters: get_model_labels_and_output_layers.py
* Object_detection.py
  * get_detection_results.py
  * detect_object
```
    convert_image_to_blob(frame)

    confidences, boxes, centroids = get_detection_details(frame,
                                                          each_layer_output,
                                                          minimum_confidence_score,
                                                          object_index)
                                                          
    indexes = apply_non_maxima_suppression(boxes, confidences, minimum_confidence_score, nms_threshold_value)
```
* detect_violations.py
```
    if len(results) >= 2:
        # Step 1: Distance Calculation based on Centroids
        # i. Create an Array of Centroids
        centroids = np.array([r[2] for r in results])

        # ii. Compute the Euclidean distances between "All pairs" of the centroids
        distance_between_observations = dist.cdist(centroids, centroids, metric="euclidean")

        # Step 2: Compare calculated distance with the Minimum distance to be flagged as Violation
        for i in range(0, distance_between_observations.shape[0]):
            for j in range(i + 1, distance_between_observations.shape[1]):

                # Step 3: Is Distance between any two centroid pairs is less than the configured "Number of pixels"
                if distance_between_observations[i, j] < minimum_distance:

                    # Step 4: Mark the observations/persons as Violations
                    set_of_violations.add(i)
                    set_of_violations.add(j)

    return set_of_violations
    
 ```
* draw_detections_and_violations.py
 
 ## Detection from the Video
```
detect_video.py
```
Same as above process, except that we process for each frame
```
    # video_stream = cv2.VideoCapture(-1)
    video_stream = cv2.VideoCapture("inputs/pedestrians.mp4")
    while True:
        # Read the next frame from the file
        (grabbed, frame) = video_stream.read()
        # End of the stream: when frame is not grabbed then we have reached the end of stream
        if not grabbed:
            break

        frame = imutils.resize(frame, width=700)
        detection_results = get_detection_results(frame,
                                                  model,
                                                  layer_numbers,
                                                  person_index
                                                  )
        # print(detection_results)
        # (0.998525857925415, (377, 179, 557, 736), (467, 458))
        # Confidence Score, Bounding Box coordinates (x1, y1, x2, y2), Centroid
        violations = detect_violations(detection_results)
        # print(violations)
        output_frame = draw_detections_and_violations(frame, detection_results, violations)
        write_and_save_video(output_frame)
```

## Non Max Suppression Comparison
### [DNN](https://github.com/sbhrwl/social_distance_violations/blob/ce5bd02ce6fd4452752d0b2fe2ce13dcb869a5ef/src/detection_opencv_dnn/core/object_detection.py#L26)
```
def apply_non_maxima_suppression(boxes, confidences, minimum_confidence, nms_threshold):
    indexes = cv2.dnn.NMSBoxes(boxes, confidences, minimum_confidence, nms_threshold)
    return indexes
    
indexes = apply_non_maxima_suppression(boxes, confidences, minimum_confidence_score, nms_threshold_value)

# Format Results
# i. Confidence score/Predicted Probability
# ii. Bounding box coordinates
# iii. Centroid
results = []
if len(indexes) > 0:
    for i in indexes.flatten():
        # Step 1: Extract bounding box coordinates
        (x, y) = (boxes[i][0], boxes[i][1])
        (w, h) = (boxes[i][2], boxes[i][3])

        # Step 2: Rewrite Bounding box coordinates
        # a. Add Width to X coordinate
        # b. Add Height to Y coordinate
        r = (confidences[i], (x, y, x + w, y + h), centroids[i])
        results.append(r)
```
### [Tensorflow](https://github.com/sbhrwl/social_distance_violations/blob/ce5bd02ce6fd4452752d0b2fe2ce13dcb869a5ef/src/detection_tensorflow_framework/core/object_detection.py#L44)
```
def apply_non_max_suppression(boxes, confidence_score, iou, score):
    boxes, scores, classes, valid_detections = tf.image.combined_non_max_suppression(
        boxes=tf.reshape(boxes, (tf.shape(boxes)[0], -1, 1, 4)),
        scores=tf.reshape(confidence_score,
                          (tf.shape(confidence_score)[0], -1, tf.shape(confidence_score)[-1])),
        max_output_size_per_class=50,
        max_total_size=50,
        iou_threshold=iou,
        score_threshold=score
    )

    return boxes, scores, classes, valid_detections
```
### Advantages of Tensorflow over OpenCV DNN
* OpenCV offers **GPU module**
* You will be the one to pick which classes and methods to use so at least **some knowledge about GPU programming** will be helpful. 
* In Tensorflow, you can **easily** use the GPU implementation by setting the number of GPUs you have or if you want to use both

## Conclusion
* Object detected using **only OpenCV** is not optimal and using TensorFlow as a framework gives you more options to explore like networks, algorithms. 
* TensorFlow is optimal at **training** part i.e. at data handling(tensors) and OpenCV is optimal in **accessing and manipulating** data (resize, crop, webcams etc.,). 
* Thus, both are used together for object detection

## References
* [OpenCV](https://towardsdatascience.com/yolo-object-detection-with-opencv-and-python-21e50ac599e9 "OpenCV")
