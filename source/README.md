##  Tech Architecture
![azure_Tarch-1](https://user-images.githubusercontent.com/72341529/172136926-6f1367aa-b531-4741-ad03-ebc4216cb043.png)


<!-- DISCUSSIONS -->

##  Code Discussions

- Face Detection

  - To detect the face in the image the person uploads, we use the [Detect With Stream API](https://docs.microsoft.com/en-us/rest/api/faceapi/face/detect-with-stream).

  - In `people\views.py` we have a function `generate_face_id` that uses the Detect With Stream API to get the faceID, which is an identifier of the face feature and will be used in [Face - Find Similar](https://docs.microsoft.com/en-us/rest/api/faceapi/face/findsimilar).

  ```py
  # function to generate face_id using Azure Face API
  def  generate_face_id(image_path):
    face_client =  FaceClient(config['ENDPOINT'], CognitiveServicesCredentials(config['KEY']))
    response_detected_face = face_client.face.detect_with_stream(
    image=open(image_path, 'rb'),
    detection_model='detection_03',
    recognition_model='recognition_04',
  )
  return response_detected_face
  ```

- Face Recognition

  - Given query face's faceID, to search the similar-looking faces from a `faceID array`, which is an array of faceIDs generated from [Detect With Stream API](https://docs.microsoft.com/en-us/rest/api/faceapi/face/detect-with-stream), we use the [Face - Find Similar API](https://docs.microsoft.com/en-us/rest/api/faceapi/face/findsimilar).

  - In `people\views.py` we have a function `find_match` that uses this API to find a match for the reported person from the list of missing people faceIDs.

  ```py
  # function to find a match for the reported person from the list of missing people using Azure Face API
  def  find_match(reported_face_id, missing_face_ids):
    face_client =  FaceClient(config['ENDPOINT'], CognitiveServicesCredentials(config['KEY']))
    matched_faces = face_client.face.find_similar(
    face_id=reported_face_id,
    face_ids=missing_face_ids
  )
  return matched_faces
  ```

- Given the fact that I was using a free account with limited number of API calls, I have built my **MVP** with these two APIs.

- In the future I plan to make use of a `faceListId` instead of the `faceID array` for the [Face - Find Similar API](https://docs.microsoft.com/en-us/rest/api/faceapi/face/findsimilar).

- The major difference between these two is that `faceID array` contains the faces created by [Face - Detect With Url](https://docs.microsoft.com/en-us/rest/api/faceapi/face/detectwithurl) or [Face - Detect With Stream](https://docs.microsoft.com/en-us/rest/api/faceapi/face/detectwithstream), which will expire at the time specified by faceIdTimeToLive after creation, which is about 86400 seconds (24 hours) by default. A `faceListId` is created by [FaceList - Create](https://docs.microsoft.com/en-us/rest/api/faceapi/facelist/create) containing persistedFaceIds that will not expire.

- Furthermore, one could also use [PersonGroup](https://docs.microsoft.com/en-us/rest/api/faceapi/persongroup) / [LargePersonGroup](https://docs.microsoft.com/en-us/rest/api/faceapi/largepersongroup) and [Face - Identify](https://docs.microsoft.com/en-us/rest/api/faceapi/face/identify) when the face number is large, the [LargeFaceList](https://docs.microsoft.com/en-us/rest/api/faceapi/largefacelist) can support up to 1,000,000 faces.

