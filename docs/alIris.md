## alIris Interface

### 초기화 및 해제
* `alIrisInit()`는 눈동자 학습 파일이 있는 경로를 입력으로 받으며, 초기화가 실패했을 경우 음수의 값을 반환한다.

```cpp
/**
 *	@brief			Initialization function
 *	@date			2016-12-01
 *	@param [in]		pcDataPath	path to the model for AlMobile
 *	@return			Success(>0), Failure (<0)
 */
int alIrisInit(const char* pcDataPath);
```

* `alIrisRelease()`는 `alIrisInit()`를 통해 할당한 메모리 등을 해제하는 역할을 수행하며, 호출하지 않는 경우 Memory leak의 문제를 일으킬 수 있다.

```cpp
/**
 *	@brief			Release function
 *	@date			2016-12-01
 *	@return			Success(>0), Failure (<0)
 */
int alIrisRelease();
```

### 눈동자 검출
* 눈동자 검출은 `alIrisDetect()`할수를 사용하며, YUV 타입의 입력 영상과 float array 타입의 landmark 정보를 이용하여 눈동자를 검출한다. 함수의 결과값은 각각의 사람에 대하여 왼쪽/오른쪽 눈에 대하여 4개의 값 (x1, y1, x2, y2)을 반환하므로, 인당 8개의 float value를 사용한다. 따라서 입력으로 넣어주는 `iris_rect`의 크기는 `8 * 검출된 사람수`가 되어야 한다.

```cpp
/**
 *	@brief			Detect Iris
 *	@date			2016-12-01
 *	@param [in]	    image               YUV image
 *	@param [in]	    landmarks           array of landmark
 *	@param [in]     num_landmarks       number of landmark
 *	@param [in]     direction           direction of image: 0, 1, 2, or 3
 *	                                    (0 deg, 90 deg,180 deg, and 270 deg, respectively)
 *	@param [in]	    width               width of image (should be bigger than height)
 *	@param [in]	    height              height of image
 *	@param [out]	pIrisRectAll        rectangle array of iris
 */
ALCHERA_MOBILE_EXPORT
void alIrisDetect(const unsigned char *image, const float *landmarks,
                  int num_landmarks, int direction, int width,
                  int height, float *iris_rects);
```

## Example

```cpp
float* alIrisDetectUsingST(const st_mobile_106_t* faces, const int num_faces,
                           unsigned char *image, const int direction,
                           const int width, const int height)
{
    const int kNumLM = 106;
    const int kNumLMx2 = 2 * kNumLM;
    const int kLMArraySize = kNumLMx2 * num_faces;

    std::unique_ptr<float[]> lm_array(new float[kLMArraySize]);

    if (num_faces > 0) {
        for (int id = 0; id < num_faces; id++) {
            const st_mobile_106_t& r = faces[id];
            const st_pointf_t *pPoints = r.points_array;

            for (int i = 0; i < kNumLM; i++) {
                lm_array[kNumLMx2 * id + 2 * i] = pPoints[i].x;
                lm_array[kNumLMx2 * id + 2 * i + 1] = pPoints[i].y;
            }
        }
    }

    float *pIrisRects = new float[num_faces * 8];
    alIrisDetect(image, lm_array.get(), kLMArraySize, direction, width, height, pIrisRects);

    return pIrisRects;
}

void rotate90degree(float *rect, int width, int height)
{
    // order: l(0), t(1), r(2), b(3)
    float left = rect[0];
    rect[0] = height - rect[3];
    rect[3] = rect[2];
    rect[2] = height - rect[1];
    rect[1] = left;
}

void rotate270degree(float *rect, int width, int height)
{
    // order: l(0), t(1), r(2), b(3)
    float left = rect[0];
    rect[0] = rect[1];
    rect[1] = width - rect[2];
    rect[2] = rect[3];
    rect[3] = width - left;
}


float* alIrisRotate(float *iris_rects, int width, int height, int num_faces, int direction)
{
    // direction { 0: 0 deg, 1: 90 deg, 2: 180 deg, 3: 270 deg }
    void (*rotate_func)(float *rect, int width, int height);

    switch (direction) {
        case 1:
            rotate_func = rotate90degree;
            break;

        case 3:
            rotate_func = rotate270degree;
            break;

        default:
            return iris_rects;
    }

    const int kNumLoops = 2 * num_faces;
    for (int i=0; i<kNumLoops; ++i) {
        rotate_func(iris_rects, width, height);
        iris_rects += 4;
    }
    return iris_rects;
}

// in actual code
{
    ...
    float* pIrisRect = alIrisDetectUsingST(pFaceArray, iFaceCount, dst.data, rotType, dst.cols, dst.rows);
    alIrisRotate(pIrisRect, dst.cols, dst.rows, iFaceCount, rotType);
    ...
}
```
