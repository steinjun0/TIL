[구글에서 codelab으로 설명한 webGPU](https://codelabs.developers.google.com/your-first-webgpu-app?hl=ko#0)
# WebGPU
1. 웹 앱에서 GPU의 기능에 액세스하기 위한 새로운 최신 API이다
2. WebGL과 차이점?  
    1. 2007년 OpenGL ES 2.0 API와 더 오래된 OpenGL API를 기반으로 만듬(구식 아키텍처)
    2. 크로스 플랫폼 방식으로 GPU 기능을 사용 설정하는 데 초점을 맞추고 사용성 개선(Direct3D 12, Metal, Vulkan이 네이티브)
3. 렌더링 기술을 지원하는 데 필요한 기능을 갖추고 있다
4. 렌더링 외의 컴퓨팅에서도 사용할 수 있다.
5. typescript에서도 완벽하게 작동한다.(@webgpu/types 정의를 import)

# 코드 순서
1. GPUAdapter를 요청해야함. 어댑터는 특정 GPU 하드웨어를 WebGPU가 표현한 것.
`navigator.gpu.requestAdapter()`
2. GPUAdapter로 device를 요청해야함. 
`adapter.requestDevice()`
3. canvas에서 GPUCanvasContext를 요청(2d, webgl 컨텍스트를 사용하여 Canvas2D 또는 WebGL 컨첵스트를 초기화하는 것과 동일)
4. context를 configure하여 기기와 연결
    1. 이 때, format 속성이 가장 중요한데, device 및 컨첵스트에서 사용해야 하는 텍스처 형식을 전달한다.
    2. 텍스처는 WebGPU가 이미지 데이터를 저장하는 데 사용하는 객체이며, 각 텍스처에는 해당 데이터가 메모리에 배치되는 방식을 GPU에게 알리는 형식이 있다.
    3. 기기에 맞는 텍스처 형식을 사용해야 가장 성능이 우수하다.
    4. WebGGPU에서 캔버스에 사용할 형식을 알려준다.
    `navigator.gpu.getPreferredCanvasFormat()`
5. GPU 명령어를 기록하기 위한 GPUCommandEncoder 생성
6. 렌더 패스를 encoder로 시작한다.
    1. 렌더 패스는 WebGPU의 모든 그리기 작업이 발생하는 것을 말한다.
    2. 각 컨트롤은 `beginRenderPass()`로 호출하고, 이 메서드는 실행된 그리기 명령어의 출력을 수신하는 텍스처를 정의한다.  반환값이 pass이다.
    3. context.getCurrentTexture()를 호출하여 컨텍스트에서 텍스처를 가져온다.
7. 렌더 패스를 pass.end()로 종료한다.
8. device.queue.submit([encoder.finish()])로 명령 버퍼를 제출한다.
    1. 이후에 캔버스 콘텐츠를 다시 업데이트하려면 새 명령어 버퍼를 기록하고 제출하여 context.getCurrentTexture()를 다시 호출하여 렌더 패스의 새 텍스처를 가져와야 한다.

# 도형 그리기
## GPU의 그리기 방식 이해하기
1. GPU는 점, 선, 삼각형(WebGPU에서 이를 primitive로 지정함)만 처리한다.
2. 점, 또는 정점은 Catesian 좌표계의 한 점을 정의하는 X, Y, Z 값으로 정의된다.
    1. 캔버스의 너비 또는 높이에 관계없이 X축의 가장 왼쪽이 -1, 오른쪽이 +1, Y축의 가장 아래쪽이 -1, 위쪽이 +1. (0,0)은 언제나 캔버스의 중심.
3. GPU는 꼭짓점 셰이더를 통해 꼭짓점을 클립 공간으로 변환한다.
    1. 이 셰이더로 꼭짓점에서 광원까지 방향도 계산할 수 있다.
    2. GPU는 픽셀을 결정한다.
4. 프레그먼트 셰이더라는 프로그램으로 각 픽셀의 색상을 계산한다.
    1. 단순한 색상 변환
    2. 복잡한 작업
        1. 반사된 태양광에 상대적인 표면의 각도 계산
        2. 안개를 통한 필터링
        3. 표면의 금속성 정도
5. 삼각형이 정정으로 정의되어야하므로, 사각형도 2개의 삼각형으로 나눠야한다.
    ``` javascript
    const vertices = new Float32Array([ // <- 틀렸음
    //   X,    Y,
    -0.8, -0.8,
    0.8, -0.8,
    0.8,  0.8,
    -0.8,  0.8,
    ]);

    const vertices = new Float32Array([
    //   X,    Y,
    -0.8, -0.8, // Triangle 1 (Blue)
    0.8, -0.8,
    0.8,  0.8,

    -0.8, -0.8, // Triangle 2 (Red)
    0.8,  0.8,
    -0.8,  0.8,
    ]);
    ```
    1. index Buffer를 사용하면 삼각형을 알아서 만들어 준다.
6. GPU는 최적화된 메모리를 사용하기 위해, 자바스크립트 배열을 GPUBuffer로 변환해야합니다.
```javascript
const vertexBuffer = device.createBuffer({
  label: "Cell vertices",
  size: vertices.byteLength,
  usage: GPUBufferUsage.VERTEX | GPUBufferUsage.COPY_DST,
});
```
    1. 라벨을 부여해야 나중에 디버깅을 할 수 있다.
7. 버퍼의 값을 변경하기 가장 쉬운 방법은 writeBuffer를 호출해서 TypedArray를 전달하는 것.
```javascript
device.queue.writeBuffer(vertexBuffer, /*bufferOffset=*/0, vertices);
```
