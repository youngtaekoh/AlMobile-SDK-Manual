# alFace Interface

![](/figs/landmark_106_index.jpg "Landmark index")  
Landmark points diagram

## Interfaces

### 초기화

```cpp
/**
 * @brief alFace Initialize
 * @param path  binary file path
 * @return returns negative value (<0) if fails to initialize
 */
al_result_t alFaceInit(const char* path);
```

### 얼굴 Landmark 검출 및 추적

이 함수에서 구해진 landmark는 반드시 `alFaceIDLandmarkDestory()`를 통해 deallocate되어야 함.

```cpp
/**
 * @brief Detect face
 * @param yuv           YUV format image
 * @param width_        width of image
 * @param height_       height of image
 * @param landmarks     pointer of the landmark (it will be allocated by this function)
 * @param fmt           image format type (support only AL_PIX_FMT_NV21 or AL_PIX_FMT_NV12)
 * @param rotation_type direction of image, support AL_CLOCKWISE_ROTATE_{0 / 90 / 180 / 270 / DEFAULT}
 *                      AL_CLOCKWISE_ROTATE_DEFAULT is automatically set to OS dependant direction of image when the device is on the upright position
 * @return number of detected faces
 * @note Use `alFaceIDLandmarkDestory()` to deallocated founded landmarks
 */
int alFaceDetect(unsigned char* yuv, int width_, int height_, alIDLandmark** landmarks, alPixelFormat fmt, RotationType rotation_type = AL_CLOCKWISE_ROTATE_DEFAULT);

/**
 * @brief Deallocates landamrks
 * @param landmarks pointer of the landmark
 * @return returns AL_SUCCESS if succeeded.
 */
al_result_t alFaceIDLandmarkDestory(alIDLandmark** landmarks);
```

### Release 함수

```cpp
/**
 * @brief Releases alFace
 * @return AL_SUCCESS (always)
 */
al_result_t alFaceRelease();
```

## Example

```cpp
#include "AlMobile.h"

// Android/iOS video frame 구조체
struct VideoFrame
{    
    size_t width;
    size_t height;
    size_t stride;

    unsigned char * data;
};

bool init()
{
    if (!alFaceInit("alTotal.bin"))
        return false;
    return true;
}

bool processVideoFrame(const VideoFrame& frame)
{
    alIDLandmark *landmarks = nullptr;
    int num_faces = alFaceDetect(frame.data, (int)frame.width, (int)frame.height, &landmarks, AL_PIX_FMT_NV12);

    if (num_faces > 0) {
        // Do whatever you want with landmarks
        alFaceIDLandmarkDestory(&landmarks);
        return true;
    }
    return false;
}
```



