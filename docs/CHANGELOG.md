# CHANGELOG
## 2017-6-19
### alFace update

## 2017-4-7
### 얼굴 검출 및 추적 모듈 alFace (prototype) 추가됨

## 2017-2-24
### 손 검출 모듈 alHand 추가됨

## 2017-1-31
### 3D deform 모듈 추가됨

## 2016-12-08
### 눈동자 검출 모듈 추가됨
 - `alIrisInit()`
 - `alIrisDetect()`
 - `alIrisRelease()`
 
### `AlMobile.h` 추가함
기존에 사용하던 `alFITLib.h`와 `common.h`를 통합하여 AlMobile.h로 통합함.
눈동자 검출 모듈을 사용하기 위해서는 AlMobile.h를 include하는 것이 필요.

## 2016-11-07
1. 입을 벌렸을 때 Object가 커지는 현상 완화
2. `alGet3DModelInfo()`에서 사용하지 않는 vertex도 반환하는 것을 고침

## 2016-10-19

### 1. Function parameter changed

- `alGet3DHeadPoseWCandideTrack()`에서 `matTransform` 및 `computeProjectionPoint` 삭제하고, parameter 위치 변경함

- `alGetTransformMat()`에서 회전각을 수정하지 않는 pure matrix를 반환해주는 `matTtransformPure` 파라미터 추가

## 2016-10-17

### 1. Function parameter changed

`matTransform`에 `nullptr`를 넣어도 작동하도록 변경함. 가급적 `alGetTransformMat()` 활용 바랍니다.

**NOTE:* _추후 버전에서는 matTransform및 computeProjectionPoint가 사라질 예정임_

### 2. Function added
Transform matrix를 faceID 별로 구할 수 있는 `alGetTransformMat()` 함수 추가함.

**NOTE:* _위 함수는 `alGet3DHeadPoseWCandideTrack()` 호출 후에 제대로 작동함._

### 3. Function modified
`alGetPerspectiveness()`, `alSetPerspectiveness()`, `alGetCameraMat()`, `alGetAnimatedModel()`, `alGetAnimationParams()`에서 0, 1, 2 ... nPerson으로 지정하던 값을 faceID를 사용하도록 변경함. 

**NOTE:* _`alGetCameraMat()`, `alGetAnimatedModel()`, `alGetAnimationParams()` 함수들은 `alGet3DHeadPoseWCandideTrack()` 호출 후에 제대로 작동함._


### 4. Function name changed

`alGetAnimatedModelPerTheFace()`를 `alGetAnimatedModel()`로 변경

