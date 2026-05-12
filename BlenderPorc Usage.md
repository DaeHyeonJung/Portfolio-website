# BlenderProc Basic Usage

## 목차

- [1. BlenderProc 개요](#1-blenderproc-개요)
- [2. 설치 방법](#2-설치-방법)
- [3. 기본 실행 구조](#3-기본-실행-구조)
- [4. Render 기본 사용법](#4-render-기본-사용법)
- [5. COCO Annotations 생성 방법](#5-coco-annotations-생성-방법)
- [6. Semantic Segmentation 생성 방법](#6-semantic-segmentation-생성-방법)
- [7. 정리](#7-정리)

---

## 1. BlenderProc 개요

**BlenderProc**는 Blender 기반의 합성 데이터 생성 파이프라인이다.  
Blender에서 만든 3D 장면을 Python 코드로 제어하여 RGB 이미지, Depth Map, Segmentation Map, COCO Annotation 등 컴퓨터 비전 학습에 필요한 데이터를 자동으로 생성할 수 있다.

BlenderProc를 사용하면 다음과 같은 데이터를 생성할 수 있다.

- RGB 렌더링 이미지
- Depth Map
- Normal Map
- Semantic Segmentation Map
- Instance Segmentation Map
- COCO Annotation
- Synthetic Dataset

기본적인 사용 흐름은 다음과 같다.

```text
3D Scene 준비
→ BlenderProc에서 Scene 로드
→ 카메라 위치 및 해상도 설정
→ 객체별 Category ID 설정
→ RGB 이미지 렌더링
→ Annotation 또는 Segmentation Map 생성
→ 결과 저장
```

공식 GitHub 저장소는 다음과 같다.

```text
https://github.com/DLR-RM/BlenderProc
```

공식 문서는 다음에서 확인할 수 있다.

```text
https://dlr-rm.github.io/BlenderProc/
```

BlenderProc는 실제 데이터를 직접 수집하거나 사람이 직접 라벨링하기 어려운 상황에서 유용하다.  
특히 3D 모델이 준비되어 있다면, 다양한 카메라 위치와 조명 조건에서 학습용 이미지를 자동으로 생성할 수 있다.

---

## 2. 설치 방법

BlenderProc는 `pip`를 이용해 설치할 수 있다.

```bash
pip install blenderproc
```

설치가 정상적으로 되었는지 확인하려면 다음 명령어를 실행한다.

```bash
blenderproc quickstart
```

BlenderProc 스크립트는 일반적인 Python 파일처럼 보이지만, 실행할 때는 보통 `python` 명령어가 아니라 `blenderproc run` 명령어를 사용한다.

```bash
blenderproc run main.py
```

일반 Python 실행 방식은 권장되지 않는다.

```bash
python main.py
```

BlenderProc에서는 다음과 같이 실행하는 것이 일반적이다.

```bash
blenderproc run main.py
```

Blender GUI를 열어서 디버깅하고 싶다면 다음 명령어를 사용할 수 있다.

```bash
blenderproc debug main.py
```

---

## 3. 기본 실행 구조

BlenderProc의 기본 코드는 보통 다음과 같은 구조를 가진다.

```python
import blenderproc as bproc

# 1. BlenderProc 초기화
bproc.init()

# 2. Blender Scene 로드
bproc.loader.load_blend("scene.blend")

# 3. 카메라 해상도 설정
bproc.camera.set_resolution(1280, 720)

# 4. 카메라 위치 및 회전 설정
cam_pose = bproc.math.build_transformation_mat(
    [0, -5, 3],
    [1.2, 0, 0]
)
bproc.camera.add_camera_pose(cam_pose)

# 5. RGB 이미지 렌더링
data = bproc.renderer.render()

# 6. 결과 저장
bproc.writer.write_hdf5("output", data)
```

각 코드의 의미는 다음과 같다.

| 코드 | 설명 |
|---|---|
| `bproc.init()` | BlenderProc 환경 초기화 |
| `bproc.loader.load_blend()` | Blender `.blend` 파일 로드 |
| `bproc.camera.set_resolution()` | 출력 이미지 해상도 설정 |
| `bproc.camera.add_camera_pose()` | 카메라 위치 및 회전 설정 |
| `bproc.renderer.render()` | RGB 이미지 렌더링 |
| `bproc.writer.write_hdf5()` | 렌더링 결과 저장 |

---

## 4. Render 기본 사용법

Render는 BlenderProc에서 가장 기본적인 기능이다.  
3D 장면을 불러오고, 카메라를 설정한 뒤 RGB 이미지를 생성하는 과정이다.

### 4.1 RGB 이미지 렌더링 예시

```python
import blenderproc as bproc

# BlenderProc 초기화
bproc.init()

# Blender Scene 로드
bproc.loader.load_blend("scene.blend")

# 이미지 해상도 설정
bproc.camera.set_resolution(1280, 720)

# 카메라 위치 및 회전 설정
cam_pose = bproc.math.build_transformation_mat(
    [0, -5, 3],
    [1.2, 0, 0]
)
bproc.camera.add_camera_pose(cam_pose)

# RGB 이미지 렌더링
data = bproc.renderer.render()

# 결과 저장
bproc.writer.write_hdf5("output/render", data)
```

---

### 4.2 출력 결과

RGB 렌더링 결과는 다음 데이터에 저장된다.

```python
data["colors"]
```

`write_hdf5()`를 사용하면 결과가 `.hdf5` 파일로 저장된다.

예시 출력 구조는 다음과 같다.

```text
output/
└── render/
    └── 0.hdf5
```

---

### 4.3 여러 카메라 위치에서 렌더링하기

하나의 장면에서 여러 카메라 위치를 설정하면 여러 시점의 이미지를 생성할 수 있다.

```python
import blenderproc as bproc

bproc.init()

bproc.loader.load_blend("scene.blend")

bproc.camera.set_resolution(1280, 720)

camera_poses = [
    ([0, -5, 3], [1.2, 0, 0]),
    ([3, -5, 3], [1.2, 0, 0.5]),
    ([-3, -5, 3], [1.2, 0, -0.5])
]

for location, rotation in camera_poses:
    cam_pose = bproc.math.build_transformation_mat(location, rotation)
    bproc.camera.add_camera_pose(cam_pose)

data = bproc.renderer.render()

bproc.writer.write_hdf5("output/multi_view_render", data)
```

위 코드에서는 카메라 위치를 3개 설정했기 때문에, 동일한 장면을 서로 다른 시점에서 렌더링할 수 있다.

---

### 4.4 Render 사용 시 주의할 점

- 해상도가 높을수록 이미지 품질은 좋아지지만 렌더링 시간이 증가한다.
- 카메라 위치와 회전값에 따라 생성되는 이미지의 구도가 크게 달라진다.
- RGB 이미지만 필요한 경우에는 `bproc.renderer.render()`만 사용해도 된다.
- 객체 탐지나 분할 학습용 데이터를 만들려면 RGB 이미지와 함께 Annotation 또는 Segmentation Map을 생성해야 한다.

---

## 5. COCO Annotations 생성 방법

COCO Annotation은 객체 탐지와 Instance Segmentation에서 많이 사용되는 데이터 형식이다.

COCO Annotation에는 보통 다음 정보가 포함된다.

- 이미지 정보
- 클래스 정보
- 객체 Bounding Box
- 객체 Segmentation Mask
- 객체 면적
- 객체 Instance ID

COCO Annotation은 일반적으로 `.json` 파일로 저장된다.

---

### 5.1 COCO Annotation 출력 구조

COCO Annotation을 생성하면 보통 다음과 같은 구조로 결과가 저장된다.

```text
output_coco/
├── images/
│   ├── 000000.png
│   ├── 000001.png
│   └── ...
└── coco_annotations.json
```

`images` 폴더에는 렌더링된 RGB 이미지가 저장되고,  
`coco_annotations.json` 파일에는 이미지별 객체 정보가 저장된다.

---

### 5.2 Category ID 설정

COCO Annotation을 생성하려면 각 객체에 `category_id`를 지정해야 한다.  
`category_id`는 해당 객체가 어떤 클래스에 속하는지를 나타낸다.

예시는 다음과 같다.

```python
for obj in bproc.object.get_all_mesh_objects():
    if obj.get_name() == "Cube":
        obj.set_cp("category_id", 1)
    elif obj.get_name() == "Sphere":
        obj.set_cp("category_id", 2)
```

위 코드에서 의미는 다음과 같다.

```text
1 = Cube
2 = Sphere
```

객체가 많아질 경우에는 Dictionary 형태로 정리하는 것이 더 깔끔하다.

```python
CATEGORY_MAP = {
    1: ["Cube"],
    2: ["Sphere"],
    3: ["Cylinder"]
}

for obj in bproc.object.get_all_mesh_objects():
    obj_name = obj.get_name()

    for category_id, object_names in CATEGORY_MAP.items():
        if obj_name in object_names:
            obj.set_cp("category_id", category_id)
```

위 코드에서 클래스 구조는 다음과 같다.

```text
1 = Cube
2 = Sphere
3 = Cylinder
```

---

### 5.3 COCO Annotation용 Segmentation Map 생성

COCO Annotation을 생성하려면 객체의 Instance 정보와 Class 정보가 필요하다.  
이를 위해 `render_segmap()`을 사용한다.

```python
seg_data = bproc.renderer.render_segmap(
    map_by=["instance", "class", "name"]
)
```

각 항목의 의미는 다음과 같다.

| 항목 | 설명 |
|---|---|
| `instance` | 객체 개별 Instance 구분 |
| `class` | 객체의 Class ID 정보 |
| `name` | 객체 이름 정보 |

COCO Annotation은 객체별 Bounding Box와 Mask 정보를 생성해야 하므로, 단순 RGB 이미지뿐 아니라 Segmentation 정보도 필요하다.

---

### 5.4 COCO Annotation 저장

COCO Annotation은 `write_coco_annotations()`를 이용해 저장할 수 있다.

```python
bproc.writer.write_coco_annotations(
    "output_coco",
    instance_segmaps=seg_data["instance_segmaps"],
    instance_attribute_maps=seg_data["instance_attribute_maps"],
    colors=data["colors"],
    color_file_format="PNG"
)
```

주요 인자는 다음과 같다.

| 인자 | 설명 |
|---|---|
| `"output_coco"` | 결과 저장 경로 |
| `instance_segmaps` | 객체 Instance Segmentation Map |
| `instance_attribute_maps` | 객체별 속성 정보 |
| `colors` | RGB 렌더링 이미지 |
| `color_file_format` | 저장할 이미지 형식 |

---

### 5.5 COCO Annotation 전체 예시 코드

```python
import blenderproc as bproc

# BlenderProc 초기화
bproc.init()

# Blender Scene 로드
bproc.loader.load_blend("scene.blend")

# 객체별 Category ID 설정
CATEGORY_MAP = {
    1: ["Cube"],
    2: ["Sphere"],
    3: ["Cylinder"]
}

for obj in bproc.object.get_all_mesh_objects():
    obj_name = obj.get_name()

    for category_id, object_names in CATEGORY_MAP.items():
        if obj_name in object_names:
            obj.set_cp("category_id", category_id)

# 카메라 설정
bproc.camera.set_resolution(1280, 720)

cam_pose = bproc.math.build_transformation_mat(
    [0, -5, 3],
    [1.2, 0, 0]
)
bproc.camera.add_camera_pose(cam_pose)

# RGB 이미지 렌더링
data = bproc.renderer.render()

# Segmentation Map 렌더링
seg_data = bproc.renderer.render_segmap(
    map_by=["instance", "class", "name"]
)

# COCO Annotation 저장
bproc.writer.write_coco_annotations(
    "output_coco",
    instance_segmaps=seg_data["instance_segmaps"],
    instance_attribute_maps=seg_data["instance_attribute_maps"],
    colors=data["colors"],
    color_file_format="PNG"
)
```

---

### 5.6 COCO Annotation 사용 시 주의할 점

#### 1. 객체 이름이 정확해야 한다

Blender 내부 객체 이름과 코드에서 작성한 이름이 다르면 `category_id`가 적용되지 않는다.

예를 들어 Blender 객체 이름이 `Cube.001`인데 코드에는 `Cube`라고 작성하면 조건문이 실행되지 않는다.

```python
if obj.get_name() == "Cube":
    obj.set_cp("category_id", 1)
```

이 경우 실제 객체 이름을 확인하고 코드에 정확히 반영해야 한다.

---

#### 2. category_id가 없는 객체는 Annotation에 제대로 포함되지 않을 수 있다

COCO Annotation을 생성할 대상 객체에는 반드시 `category_id`가 설정되어 있어야 한다.

```python
obj.set_cp("category_id", 1)
```

---

#### 3. COCO와 YOLO의 Class ID 체계가 다를 수 있다

COCO Annotation에서는 Class ID가 1부터 시작하는 경우가 많지만, YOLO 형식에서는 보통 0부터 시작한다.

예를 들어 COCO에서는 다음과 같을 수 있다.

```text
1 = Cube
2 = Sphere
```

YOLO 형식으로 변환하면 다음처럼 바뀔 수 있다.

```text
0 = Cube
1 = Sphere
```

따라서 COCO Annotation을 다른 형식으로 변환할 때는 Class ID가 밀리지 않았는지 확인해야 한다.

---

## 6. Semantic Segmentation 생성 방법

Semantic Segmentation은 이미지의 각 픽셀을 특정 클래스에 할당하는 작업이다.

객체 탐지는 다음 질문에 답한다.

```text
객체가 어디에 있는가?
```

Semantic Segmentation은 다음 질문에 답한다.

```text
각 픽셀이 어떤 클래스에 속하는가?
```

예를 들어 Segmentation Mask의 픽셀 값이 다음과 같을 수 있다.

```text
0 = Background
1 = Cube
2 = Sphere
3 = Cylinder
```

즉, 픽셀 값이 `1`이면 Cube 영역이고, 픽셀 값이 `2`이면 Sphere 영역이다.

---

### 6.1 Semantic Segmentation용 Category ID 설정

Semantic Segmentation에서도 각 객체가 어떤 클래스에 속하는지 알아야 한다.  
따라서 COCO Annotation과 마찬가지로 `category_id`를 설정한다.

```python
for obj in bproc.object.get_all_mesh_objects():
    if obj.get_name() == "Cube":
        obj.set_cp("category_id", 1)
    elif obj.get_name() == "Sphere":
        obj.set_cp("category_id", 2)
```

객체가 많을 경우 다음처럼 Dictionary를 사용할 수 있다.

```python
CATEGORY_MAP = {
    1: ["Cube"],
    2: ["Sphere"],
    3: ["Cylinder"]
}

for obj in bproc.object.get_all_mesh_objects():
    obj_name = obj.get_name()

    for category_id, object_names in CATEGORY_MAP.items():
        if obj_name in object_names:
            obj.set_cp("category_id", category_id)
```

---

### 6.2 Semantic Segmentation Map 생성

Semantic Segmentation Map은 `render_segmap()`을 사용해 생성할 수 있다.

```python
seg_data = bproc.renderer.render_segmap(
    map_by=["class"]
)
```

여기서 `"class"`는 객체에 설정된 `category_id`를 기준으로 픽셀별 클래스 맵을 생성한다.

생성된 Semantic Segmentation Map은 다음 데이터에 저장된다.

```python
seg_data["class_segmaps"]
```

각 픽셀 값은 해당 픽셀이 속한 클래스 ID를 의미한다.

예를 들어 다음과 같은 클래스 구조라면,

```text
0 = Background
1 = Cube
2 = Sphere
3 = Cylinder
```

Segmentation Map에서 픽셀 값이 `2`인 영역은 Sphere에 해당한다.

---

### 6.3 Semantic Segmentation 전체 예시 코드

```python
import blenderproc as bproc

# BlenderProc 초기화
bproc.init()

# Blender Scene 로드
bproc.loader.load_blend("scene.blend")

# 객체별 Category ID 설정
CATEGORY_MAP = {
    1: ["Cube"],
    2: ["Sphere"],
    3: ["Cylinder"]
}

for obj in bproc.object.get_all_mesh_objects():
    obj_name = obj.get_name()

    for category_id, object_names in CATEGORY_MAP.items():
        if obj_name in object_names:
            obj.set_cp("category_id", category_id)

# 카메라 설정
bproc.camera.set_resolution(1280, 720)

cam_pose = bproc.math.build_transformation_mat(
    [0, -5, 3],
    [1.2, 0, 0]
)
bproc.camera.add_camera_pose(cam_pose)

# RGB 이미지 렌더링
data = bproc.renderer.render()

# Semantic Segmentation Map 생성
seg_data = bproc.renderer.render_segmap(
    map_by=["class"]
)

# RGB 이미지와 Semantic Segmentation Map 저장
bproc.writer.write_hdf5(
    "output_semantic",
    {
        "colors": data["colors"],
        "class_segmaps": seg_data["class_segmaps"]
    }
)
```

---

### 6.4 Semantic Segmentation 출력 결과

예시 출력 구조는 다음과 같다.

```text
output_semantic/
└── 0.hdf5
```

저장된 데이터에는 다음 정보가 포함된다.

```text
colors        = RGB 이미지
class_segmaps = Semantic Segmentation Map
```

`class_segmaps`는 픽셀 단위의 클래스 ID를 가진다.

예시는 다음과 같다.

```text
0 = Background
1 = Cube
2 = Sphere
3 = Cylinder
```

---

### 6.5 Semantic Segmentation과 Instance Segmentation의 차이

Semantic Segmentation과 Instance Segmentation은 비슷해 보이지만 목적이 다르다.

Semantic Segmentation은 같은 클래스의 객체를 같은 라벨로 처리한다.

```text
이미지 안에 Cube가 2개 있어도 둘 다 class 1로 표시된다.
```

Instance Segmentation은 같은 클래스라도 객체 개체별로 구분한다.

```text
Cube 1과 Cube 2를 서로 다른 객체로 구분한다.
```

비교하면 다음과 같다.

| 구분 | 의미 | 예시 |
|---|---|---|
| Semantic Segmentation | 픽셀별 클래스 구분 | 모든 Cube = class 1 |
| Instance Segmentation | 픽셀별 객체 개체 구분 | Cube 1, Cube 2를 따로 구분 |
| COCO Annotation | 객체 탐지 및 Instance 정보 저장 | Bounding Box + Mask + Category |

---

### 6.6 Semantic Segmentation 사용 시 주의할 점

#### 1. Mask 값은 색상이 아니라 Class ID이다

Segmentation Mask를 시각화하면 색깔로 보일 수 있지만, 실제 학습에 사용하는 값은 색상이 아니라 픽셀의 Class ID이다.

예를 들어 다음과 같다.

```text
0 = Background
1 = Cube
2 = Sphere
```

따라서 시각화용 이미지와 학습용 Mask를 혼동하지 않아야 한다.

---

#### 2. Background는 보통 0으로 처리한다

대부분의 Semantic Segmentation 학습에서는 배경 클래스를 `0`으로 사용한다.

```text
0 = Background
1 = Object Class 1
2 = Object Class 2
```

---

#### 3. 객체 이름과 category_id를 반드시 확인해야 한다

객체 이름이 코드와 맞지 않으면 해당 객체의 Category ID가 설정되지 않는다.  
그러면 Segmentation Map에서 해당 객체가 원하는 클래스로 표시되지 않을 수 있다.

---

## 7. 정리

BlenderProc는 Blender 장면을 기반으로 컴퓨터 비전 학습용 합성 데이터를 생성할 수 있는 도구이다.

기본 흐름은 다음과 같다.

```text
Blender Scene 로드
→ 카메라 위치 설정
→ 객체 Category ID 설정
→ RGB 이미지 렌더링
→ Annotation 또는 Segmentation Map 생성
→ 결과 저장
```

이 문서에서는 세 가지 기본 사용법을 정리하였다.

| 항목 | 목적 | 주요 출력 |
|---|---|---|
| Render | RGB 이미지 생성 | 렌더링 이미지 또는 HDF5 |
| COCO Annotations | 객체 탐지 및 Instance Segmentation용 Annotation 생성 | JSON + RGB 이미지 |
| Semantic Segmentation | 픽셀 단위 클래스 Label 생성 | Class Segmentation Map |

각 기능의 핵심은 다음과 같다.

```text
Render:
3D 장면을 RGB 이미지로 렌더링한다.

COCO Annotations:
객체별 Bounding Box와 Mask 정보를 JSON 형식으로 저장한다.

Semantic Segmentation:
각 픽셀의 Class ID를 가진 Segmentation Map을 생성한다.
```

BlenderProc에서 Annotation과 Segmentation을 생성할 때 가장 중요한 개념은 `category_id`이다.

```python
obj.set_cp("category_id", 1)
```

`category_id`는 Blender 객체와 머신러닝 클래스 라벨을 연결하는 역할을 한다.  
따라서 객체 이름, Category ID, 저장 형식을 정확히 관리해야 원하는 데이터셋을 생성할 수 있다.