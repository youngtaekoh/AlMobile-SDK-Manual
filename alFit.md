# alFit: Head pose 추정 모듈

## Interface

* ### 초기화

```cpp
/**    
 *    @brief            Initialization function
 *    @date            2016-06-29
 *    @param [in]        pcDataPath        folder path with all paremeters and models for Alchera Face InTelligence System
 *    @return            Success(>0), Failure (<0)
 */
int alInitFIT(const char* pcDataPath);
```

* ### 3차원 포즈 및 표정 변화 추적

Head pose matrix는 `alGet3DHeadPoseWCandideTrack()`을 실행한 이후, `alGetTransformMat()`을 이용하여 구한다.

```cpp
/**
 *    @brief            Estimate 3D head pose
 *    @date             2016-10-19
 *    @param [in]       alflandmark             2d landmark
 *    @param [in]       nFaces                  the number of faces
 *    @param [in]       nwid                    width of input image
 *    @param [in]       nhei                    height of input image
 *    @param [in]       froty                   Pose adjustment prarameters ( radian : Default 0.18)
 *    @param [out]      vecposed2Dpts           2D projected points from 3D vertice
 *    @param [in]       computeProjectionPoint  compute vecposed2Dpts or not.
 *    @return           average of euclidean distance btw landmarks and 2d projected points
 */
float alGet3DHeadPoseWCandideTrack(const alIDLandmark * alflandmark,
                                   const int nFaces, int nwid, int nhei,
                                   bool doPoseOnly = false,
                                   float froty = 0.18f,
                                   alfPoint2D** vecposed2Dpts = nullptr);

/**
 *  @brief          Get transform matrix
 *  @date           2016-10-17
 *  @param [in]     nFaceID             The face id number from a face tracker
 *  @param [out]    matTransform        Transform matrix (in row-major manner)
 *  @param [out]    matTransformPure    Transform matrix before adjustment (in row-major manner)
 *  @return                             The i-th person in order (0th, 1st, 2nd, etc)==> NOT PERSON ID!!
 */
int alGetTransformMat(const int nFaceID, float matTransform[16], float *matTtransformPure = nullptr);
```

* ### Camera matrix를 돌려주는 함수

FaceID는 Face Tracker에서 돌려주는 Face ID value.

```cpp
/**
 *    @brief            Extracting Camera Intrinsic matrix
 *    @date            2016-07-12
 *    @param [in]        nwid                width of input image
 *    @param [in]        nhei                height of input image
 *    @param [out]    camMat                9 array camera matrix (the memory must be allocated before feeding into this function)
 *  @param [in]     nFaceID              face ID
 *    @return            Success(>0), Failure (<0)
 */
AL_FIT_EXPORT
int alGetCameraMat(const int nwid, const int nhei, float *camMat, const unsigned int nFaceID);
```

* ### Face model에 대한 Vertex와 Face list를 돌려주는 함수

```cpp
/**
 *
 *    @brief            Get 3D Model Info
 *    @date            2016-07-26
 *    @param [out]    vecfmodel            meanshape
 *    @param [out]    vecnface            triangulation index
 *    @return            Success(>0), Failure (<0)
 */
AL_FIT_EXPORT
void alGet3DModelInfo(std::vector<alfVertex> &vecfmodel, std::vector<alFace> &vecnface);
```

* ### 표정 변화를 추적하는 경우, 변형된 Face model의 vertex를 돌려주는 함수

```cpp
/**
 * @brief Get animation parameter
 *
 * @param [in]  nFaceID        face id
 * @param [out] paramAnim animation parameters
 *              paramAnim[0] : Upper lip raiser
 *              paramAnim[1] : Jaw Droper
 *              paramAnim[2] : Lip Stretcher
 *              paramAnim[3] : Brow lowerer
 *              paramAnim[4] : Lip corner depressor
 *              paramAnim[5] : Outer brow raiser
 *              paramAnim[6] : Nose wrinkler
 */
AL_FIT_EXPORT
void alGetAnimationParams(int nFaceID, std::vector<float>& vecfparamAnim, bool bNormalized = true);

/**
 *    @brief            Get 3D Animated Shapes for each face ID
 *    @date            2016-10-17
 *    @param [int]    nFaceID            face ID
 *    @param [out]    pfmodel            animated shape vertices (size : [the number of vertices])
 */
AL_FIT_EXPORT
int alGetAnimatedModel(int nFaceID, alfVertex *pfmodel);
```

* ### 현재 AlMobile의 버전을 돌려주는 함수

```cpp
/**
 *
 * @brief           Get version of this library
 * @date            2016-7-19
 * @return          Version string
 */
AL_FIT_EXPORT
const char* alGetVersion();
```

* ### 흔들림을 잡아주는 Filter를 적용하는 함수

```cpp
/**
 *    @brief            Initialize Smoothen landmark points
 *    @date            2016-08-16
 *    @param [in]        weight                weight of previous point
 *    @return            return false if not initialized.
 */
AL_FIT_EXPORT
bool alSetLMFilter(float weight);

/**
 *    @brief            Smoothen landmark points
 *    @date            2016-08-11
 *    @param [inout]    Landmark            landmark points array
 *    @param [in]        nFaces                number of person
 *    @return            return false if not initialized.
 */
AL_FIT_EXPORT
bool alSmoothenLM(alIDLandmark *Landmark, int nFaces);
```

* ### Landmark 회전 및 Mirroring 관련 함수

Front/Back camera 대응용

```cpp
/**
 *    @brief            Rotate landmark
 *    @date            2016-08-16
 *    @param [inout]    Landmark            landmark points array
 *    @param [in]        nFaces                number of faces
 *    @param [in]        width                width of image
 *    @param [in]        height                height of image
 *    @param [in]        rotate270            true (270), false(90)
 *    @return            void
 */
AL_FIT_EXPORT
void alRotate(alIDLandmark *curLM, int nFaces, int width, int height, bool rotate270);

/**
 *    @brief            Flip landmark
 *    @date            2016-08-16
 *    @param [inout]    Landmark            landmark points array
 *    @param [in]        nFaces                number of face
 *    @param [in]        width                width of image
 */
AL_FIT_EXPORT
void alMirror(alIDLandmark *curLM, int nFaces, int width);
```

* ### Perspectiveness 관련 함수

```cpp
/**
 *  @brief          Set perspectiveness
 *  @date           2016-09-07
 *  @param [in]     nFaceID             the face ID (0, 1, ..., 36)
 *  @param [in]     perspectiveness     perspectiveness value
 *  @return         return false if it is not initialized, or index is out of bound.
 */
bool alSetPerspectiveness(const unsigned int nFaceID, const float perspectiveness);

/**
 *  @brief          Get perspectiveness
 *  @date           2016-09-07
 *  @param [in]     nFaceID             the face ID
 *  @return         return negative if it is not initialized, 
 *                  or returns 1.0 (default value) if index is out of bound.
 */
float alGetPerspectiveness(const unsigned int nFaceID);
```

## Example

```cpp
#include "AlMobile.h"
#include <vector>

// Headpose matrix를 저장할 구조체    
struct Headpose {
    float camera_matrix[9];             // Camera matrix
    float headpose_matrix[16];          // Headpose matrix (보정 적용 O)
    float headpose_matrix_pure[16];     // Headpose matrix (보정 적용 X)
};

// Android/iOS video frame 구조체
struct VideoFrame
{    
    size_t width;
    size_t height;
    size_t stride;

    unsigned char * data;
};

std::vector<Headpose> headposes;

bool init()
{
    if (!alFaceInit("alTotal.bin"))
        return false;

    if (!alInitFIT(nullptr))
        return false;
    return true;
}

bool processVideoFrame(const VideoFrame& frame)
{
    alIDLandmark *landmarks = nullptr;
    int num_faces = alFaceDetect(frame.data, (int)frame.width, (int)frame.height, &landmarks, AL_PIX_FMT_NV12);

    if (num_faces > 0)
    {
        headposes.resize(num_faces);

        alMirror(landmarks, num_faces, (int)frame.height);

        alSmoothenLM(landmarks, num_faces);

        float fdist = alGet3DHeadPoseWCandideTrack(landmarks, num_faces, (int)frame.height, (int)frame.width);
        (void)fdist;

        for (int i=0; i<num_faces_; ++i)
        {
            int faceID = landmarks[i].id;
            Headpose& hp = headposes.at(i);

            alGetCameraMat(frame.height, frame.width, hp.camera_matrix, faceID);
            alGetTransformMat(faceID, hp.headpose_matrix, hp.headpose_matrix_pure);
        }
        alFaceIDLandmarkDestory(&landmarks);
        return true;
    }
    return false;
}

void compute_projection_matrix(const Headpose& hp, float projectionMatrix[16])
{
    const float far = 100;            // 사용자 설정값
    const float near = 0.01;          // 사용자 설정값
    const float fx = hp.camera_matrix[0];
    const float fy = hp.camera_matrix[4];
    const float cx = hp.camera_matrix[2];
    const float cy = hp.camera_matrix[5];
    float m[16] = {fx/cx, 0, 0, 0,
                    0, fy/cy, 0, 0,
                    0, 0, -(far+near)/(far-near), -2*far*near/(far-near),
                    0, 0, -1, 0};
    memcpy(projectionMatrix, m, sizeof(float) * 16);
}
```



