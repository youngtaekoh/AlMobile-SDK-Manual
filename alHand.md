## alHand Interface

![][fig_hand_ex]

[fig_hand_ex]: figs/hand_example.png

### 초기화 및 해제
* alHand는 기본적으로 YUV 타입의 입력 영상을 이용하여 손 형상를 검출한다. (현재는 `AL_PIX_FMT_NV21`, `AL_PIX_FMT_NV12`만 지원함), 초기화가 실패했을 경우 음수의 값을 반환한다. Hand detection type은 가위바위보(AL_HAND_ROCK_PAPER_SCISSORS) 및 미니하트(AL_HAND_MINI_HEART)를 지원한다.

```cpp
/**
 *	@brief			alHand Initialization function
 *	@date			2017-02-23
 *	@param [in]	    width       width of image (should be bigger than height)
 *	@param [in]	    height      height of image
 *	@param [in]     pixelFormat pixel format (currently supports only AL_PIX_FMT_NV21 and AL_PIX_FMT_NV12)
 *	@param [in]     type        Hand detection type
 *	@return			Success(>0), Failure (<0)
 */
int alHandInit(int width, int height, const alPixelFormat pixelFormat, HandDetectionType type = AL_HAND_ROCK_PAPER_SCISSORS);
```

* `alHandRelease()`는 `alHandInit()`를 통해 할당한 메모리 등을 해제하는 역할을 수행하며, 호출하지 않는 경우 Memory leak의 문제를 일으킬 수 있다.

```cpp
/**
 *	@brief			Hand detection release function
 *	@date			2017-02-23
 *	@return			Success(>0), Failure (<0)
 */
int alHandRelease();
```

### 손 검출
* 손 검출은 `alHandDetect()`할수를 사용하며,  함수의 결과값은 각각의 손에 대하여 6개의 값 (class, tracking id, left, top, right, bottom)을 반환하므로, 손 하나당 6개의 float value를 사용한다. 따라서 입력으로 넣어주는 `pHandRectAll`의 크기는 `6 * 검출된 사람수`가 되어야 한다. 검출될 손의 수는 10개를 넘지 않는다. (maxHand <= 10)

```cpp
/**
 *	@brief			Detect Hand
 *	@date			2017-02-23
 *	@param [in]		image               input image
 *	@param [in]		direction           direction of image
 *	@param [in]     maxHand             Maximum number of hands to be detected. (<=10)
 *	@param [out]	pHandRectAll        { class, tracking id, rectangle(l, t, r, b) } array of hand
 */
int alHandDetect(const unsigned char* image, const RotationType direction, const int maxHand, float* pHandRectAll);
```

* Tracking sensitivity는 아래 함수를 통해 설정할 수 있으며, 기본값은 0.5이다. 숫자가 커질수록 민감도가 증가하나, 값의 범위는 0-1이며, 이를 벗어나는 값은 가장 근사한 값으로 자동 설정된다.
```cpp
/**
 *	@brief			Set tracking sensitivity
 *	@param [in]	    value               tracking sensitivity (0~1), 1 means most sensitive, 0 means least sensitive.
 *	                                    0.5 is default value
 *	@return			Success(>0), Failure (<0)
 */
int alHandSetTrackSensitivity(const int value);
```

## Example

```cpp
alHandInit(width, height, AL_PIX_FMT_NV12, AL_HAND_ROCK_PAPER_SCISSORS);
...
std::vector<float> hand_det_result_(6 * 5);
int num_hands = alHandDetect(yuv.data(), rotType, 5, hand_det_result_.data());
...
alHandRelease();
```