<!-- 想做一个过马路的应用 -->
<script setup lang="ts">
import { reactive, ref, onMounted } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { Texture } from 'three';
let dialogVisable = ref(false);
let boardText: any = ref('');
const init = () => {
  // 创建场景
  const scene = new THREE.Scene();
  let axes = new THREE.AxesHelper(100);
  // 坐标系添加在场景中
  scene.add(axes);
  // 创建摄像头
  const camera = new THREE.OrthographicCamera(
    window.innerWidth / -2,
    window.innerWidth / 2,
    window.innerHeight / 2,
    window.innerHeight / -2,
    0.1,
    10000
  );
  4;
  const distance = 500; //初始化相机位置
  const initialCameraPositionY = -Math.tan(camera.rotation.x) * distance;
  const initialCameraPositionX =
    Math.tan(camera.rotation.y) *
    Math.sqrt(distance ** 2 + initialCameraPositionY ** 2);
  camera.position.y = initialCameraPositionY;
  camera.position.x = initialCameraPositionX;
  camera.position.z = distance;

  camera.rotation.x = (65 * Math.PI) / 180; //相机视角转换，顺时针 右x：50度，上y：20度，外z：10度
  camera.rotation.y = (25 * Math.PI) / 180;
  camera.rotation.z = (5 * Math.PI) / 180;
  // 小鸡的尺寸
  const chickenSize = 15;
  // 放大
  const zoom = 2;
  const positionWidth = 42;
  const columns = 17;

  class Texture {
    constructor(
      width: number,
      height: number,
      rects: { x: number; y: number; w: number; h: number }[]
    ) {
      // 创建canvas 对象
      const canvas = document.createElement('canvas');
      // 设置canvas对象的宽度 和 高度
      canvas.width = width;
      canvas.height = height;
      const context = canvas.getContext('2d') as CanvasRenderingContext2D;
      context.fillStyle = '#ffffff';
      context.fillRect(0, 0, width, height);
      context.fillStyle = 'rgba(0,0,0,0.5)';
      rects.forEach((rect) => {
        context.fillRect(rect.x, rect.y, rect.w, rect.h);
      });
      return new THREE.CanvasTexture(canvas);
    }
  }
  // 绘制广告文字
  class Boardtxt {
    constructor(width: number, height: number, str: string) {
      const canvas = document.createElement('canvas');
      // 设置canvas对象的宽度 和 高度
      canvas.width = width;
      canvas.height = height;
      const context = canvas.getContext('2d') as CanvasRenderingContext2D;
      // context.fillStyle = '#dd33dd';
      // context.fillRect(0, 0, width, height);
      context.textAlign = 'center';
      context.strokeText(str, width / 2, height / 2);

      return new THREE.CanvasTexture(canvas);
    }
  }
  // 小汽车的车窗纹理
  const carFrontTexture: any = new Texture(40, 80, [
    { x: 0, y: 10, w: 30, h: 60 },
  ]);
  const carBackTexture: any = new Texture(40, 80, [
    { x: 10, y: 10, w: 30, h: 60 },
  ]);
  const carRightSideTexture: any = new Texture(110, 40, [
    { x: 10, y: 0, w: 50, h: 30 },
    { x: 70, y: 0, w: 30, h: 30 },
  ]);
  const carLeftSideTexture: any = new Texture(110, 40, [
    { x: 10, y: 10, w: 50, h: 30 },
    { x: 70, y: 10, w: 30, h: 30 },
  ]);

  const treeHight = [20, 30, 40]; // 树的高度
  const laneSpeeds = [2, 2.5, 3]; // 车辆的移动速度
  const vechicleColors = [0xa52523, 0xbdb638, 0x78b14b]; // 车厢的颜色
  const laneTypes = ['car', 'truck', 'forest']; // 车道类型
  const boardWidth = positionWidth * columns;
  let chooseMesh = null; // 已选中模型
  // 鼠标选中模型列表
  let intersectsArr: any = [];
  class Chicken {
    constructor() {
      const chicken = new THREE.Group(); // group 是一个多个网格的组
      // 创建 身体
      const bodyGeometry = new THREE.BoxBufferGeometry(
        chickenSize * zoom,
        chickenSize * zoom,
        chickenSize * zoom
      );
      const bodyMaterial = new THREE.MeshPhongMaterial({
        color: 0xffffff,
        flatShading: true,
      });
      const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
      body.position.z = 10 * zoom;
      body.castShadow = true;
      body.receiveShadow = true;
      chicken.add(body);
      // 创建鸡冠
      const rowelgeometry = new THREE.BoxBufferGeometry(
        2 * zoom,
        4 * zoom,
        2 * zoom
      );
      const rowelMaterial = new THREE.MeshPhongMaterial({
        color: 0xff0000,
        flatShading: true,
      });
      const rowel = new THREE.Mesh(rowelgeometry, rowelMaterial);
      rowel.position.z = 21 * zoom;
      rowel.castShadow = true;
      rowel.receiveShadow = false;
      chicken.add(rowel);
      return chicken;
    }
  }

  class Tree {
    constructor() {
      const tree = new THREE.Group();
      // 树枝
      let num = Math.floor(Math.random() * 3);
      const bole = new THREE.Mesh(
        new THREE.BoxBufferGeometry(20, 20, 20),
        new THREE.MeshPhongMaterial({
          color: 0x412320,
          flatShading: true,
        })
      );
      bole.position.z = 5 * zoom;
      bole.position.x = 0;
      bole.position.y = 0;
      bole.castShadow = true;
      bole.receiveShadow = true;
      tree.add(bole);

      // 获得一个 0 到3 的随机数
      const branch = new THREE.Mesh(
        new THREE.BoxBufferGeometry(
          20 * zoom,
          20 * zoom,
          treeHight[num] * zoom
        ),
        new THREE.MeshPhongMaterial({
          color: 0x216e39,
          flatShading: true,
        })
      );
      branch.position.z = (treeHight[num] / 2 + 10) * zoom;
      // 树干
      tree.add(branch);

      return tree;
    }
  }
  // 草坪实体模型
  class Grass {
    constructor() {
      const grass = new THREE.Group();
      const createSection = (color: number) =>
        new THREE.Mesh(
          new THREE.BoxBufferGeometry(
            boardWidth * zoom,
            positionWidth * zoom,
            3 * zoom
          ),
          new THREE.MeshPhongMaterial({ color })
        );

      const middle = createSection(0xbaf455); //草地中间亮
      middle.receiveShadow = true;
      grass.add(middle);
      const left = createSection(0x99c846); //两侧暗
      left.position.x = -boardWidth * zoom;
      grass.add(left);
      const right = createSection(0x99c846);
      right.position.x = boardWidth * zoom;
      grass.add(right);
      grass.position.z = 1.5 * zoom;
      return grass;
    }
  }
  // 马路模型实体
  class Road {
    constructor() {
      const road = new THREE.Group();
      // 创建平面
      const createSection = (color: number) =>
        new THREE.Mesh(
          new THREE.PlaneGeometry(boardWidth * zoom, positionWidth * zoom),
          new THREE.MeshPhongMaterial({ color })
        );
      const left = createSection(0x393d49);
      left.position.x = -boardWidth * zoom;
      road.add(left);
      const middle = createSection(0x454a59);
      road.add(middle);
      const right = createSection(0x393d49);
      right.position.x = boardWidth * zoom;
      road.add(right);
      return road;
    }
  }
  class Tyre {
    constructor() {
      const tyre = new THREE.Mesh(
        new THREE.CylinderGeometry(6 * zoom, 6 * zoom, 30 * zoom, 15 * zoom),
        new THREE.MeshBasicMaterial({ color: 0x000000 })
      );
      return tyre;
    }
  }
  // 卡车模型实体
  class Truck {
    constructor() {
      // 创建卡车组
      const truck = new THREE.Group();
      // 创建卡车车头 方形
      const truckHead = new THREE.Mesh(
        new THREE.BoxBufferGeometry(30 * zoom, 30 * zoom, 30 * zoom),
        new THREE.MeshPhongMaterial({
          color: 0xdddddd,
        })
      );
      truck.position.z = 20 * zoom;
      truck.add(truckHead);

      // 创建卡车车身
      // 创建卡车车轮 3*个
      const frontTyre: any = new Tyre();
      const middleTyre: any = new Tyre();
      const backType: any = new Tyre();
      // 设置三个轮子的X轴 的位置
      frontTyre.position.x = 22 * zoom;
      frontTyre.position.z = -12 * zoom;
      middleTyre.position.x = 4 * zoom;
      middleTyre.position.z = -12 * zoom;
      backType.position.x = -22 * zoom;
      backType.position.z = -12 * zoom;
      truck.add(frontTyre);
      truck.add(middleTyre);
      truck.add(backType);

      return truck;
    }
  }
  // 小汽车模型实体
  class Car {
    constructor() {
      const car = new THREE.Group();
      // 车顶
      const cabin = new THREE.Mesh( //网格模型，车顶，统一灰色
        new THREE.BoxBufferGeometry(33 * zoom, 24 * zoom, 12 * zoom),
        [
          //材质
          new THREE.MeshPhongMaterial({
            color: 0xcccccc,
            flatShading: true,
            map: carBackTexture,
          }),
          new THREE.MeshPhongMaterial({
            color: 0xcccccc,
            flatShading: true,
            map: carFrontTexture,
          }),
          new THREE.MeshPhongMaterial({
            color: 0xcccccc,
            flatShading: true,
            map: carRightSideTexture,
          }),
          new THREE.MeshPhongMaterial({
            color: 0xcccccc,
            flatShading: true,
            map: carLeftSideTexture,
          }),
          new THREE.MeshPhongMaterial({ color: 0xcccccc, flatShading: true }), // top
          new THREE.MeshPhongMaterial({ color: 0xcccccc, flatShading: true }),
        ] // bottom
      );
      cabin.position.z = (12 + 7 + 5) * zoom;
      cabin.position.x = -5 * zoom;
      cabin.castShadow = true;
      cabin.receiveShadow = true;
      car.add(cabin);
      // 车身
      const carbody = new THREE.Mesh(
        new THREE.BoxBufferGeometry(66 * zoom, 30 * zoom, 14 * zoom),
        new THREE.MeshPhongMaterial({
          color:
            vechicleColors[Math.floor(Math.random() * vechicleColors.length)],
        })
      );
      // bot
      carbody.position.z = (7 + 5) * zoom;
      carbody.castShadow = true;
      carbody.receiveShadow = true;
      car.add(carbody);

      const frontTyre: any = new Tyre();
      frontTyre.position.z = 8 * zoom;
      frontTyre.position.x = 18 * zoom;

      const backTyre: any = new Tyre();
      backTyre.position.z = 8 * zoom;
      backTyre.position.x = -18 * zoom;
      car.add(frontTyre);
      car.add(backTyre);

      return car;
      // 车轱辘
    }
  }

  // 广告牌模型实体

  class Board {
    constructor(boardText: string) {
      // 创建一个组
      const board = new THREE.Group();
      // 广告牌上半部分  需要显示图片 或文字内容
      // 下半部分为底座
      const boardBody = new THREE.Mesh(
        new THREE.BoxBufferGeometry(80 * zoom, 20 * zoom, 50 * zoom),
        [
          new THREE.MeshPhongMaterial({ color: 0x000000, flatShading: true }),
          new THREE.MeshPhongMaterial({ color: 0x000000, flatShading: true }),
          new THREE.MeshPhongMaterial({ color: 0x000000, flatShading: true }),
          new THREE.MeshPhongMaterial({
            color: 0x000000,
            flatShading: true,
            map: new Boardtxt(80, 50, boardText) as any,
          }),
          new THREE.MeshPhongMaterial({ color: 0x000000, flatShading: true }),
          new THREE.MeshPhongMaterial({ color: 0x000000, flatShading: true }),
        ]
        // new THREE.MeshPhongMaterial({ color: 0x000000 })
      );
      boardBody.position.z = 40 * zoom;
      boardBody.castShadow = true;
      boardBody.receiveShadow = true;
      board.add(boardBody);

      const boardBottom = new THREE.Mesh(
        new THREE.BoxBufferGeometry(20 * zoom, 5 * zoom, 32 * zoom),
        new THREE.MeshPhongMaterial({ color: 0xdddddd })
      );
      boardBottom.position.z = 8 * zoom;
      boardBottom.castShadow = true;
      boardBottom.receiveShadow = true;
      board.name = boardText;
      board.add(boardBottom);

      intersectsArr.push(board);
      return board;
    }
  }
  const chicken: any = new Chicken();
  chicken.position.x = 0; //小鸡位置初始化为0
  chicken.position.y = 0;
  scene.add(chicken);

  const tree: any = new Tree();
  tree.position.x = 50; //小鸡位置初始化为0
  tree.position.y = -20;
  scene.add(tree);
  const tree2: any = new Tree();
  tree2.position.x = 100; //小鸡位置初始化为0
  tree2.position.y = -20;
  scene.add(tree2);
  const tree3: any = new Tree();
  tree3.position.x = 150; //小鸡位置初始化为0
  tree3.position.y = -20;
  scene.add(tree3);

  const grass: any = new Grass();
  scene.add(grass);

  const road: any = new Road();
  road.position.y = positionWidth * zoom;
  scene.add(road);

  const board: any = new Board('智慧车站引导屏');
  board.position.x = -100;
  board.position.y = -21 + 20;
  board.position.z = 20;
  scene.add(board);
  const board2: any = new Board('手势识别屏');
  board2.position.x = 500;
  board2.position.y = -21 + 20;
  board2.position.z = 20;
  scene.add(board2);

  const car: any = new Car();
  car.position.x = 180;
  car.position.y = -21 + 20;
  car.position.z = 0;
  scene.add(car);

  const truck: any = new Truck();
  truck.position.x = -300;
  truck.position.y = 0;
  scene.add(truck);
  // 创建构造器
  const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
  renderer.setClearColor(new THREE.Color(0xdddddd));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.render(scene, camera);
  renderer.setSize(window.innerWidth, window.innerHeight);

  const hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 0.6);
  scene.add(hemiLight);
  //平行光，太阳光（光照颜色，强度），位置，有动态阴影（平行相机）
  const dirLight = new THREE.DirectionalLight(0xffffff, 0.6);
  dirLight.position.set(-200, -100, 200);
  dirLight.castShadow = true;
  let d = 2500;
  dirLight.shadow.camera.left = -d;
  dirLight.shadow.camera.right = d;
  dirLight.shadow.camera.top = d;
  dirLight.shadow.camera.bottom = -d;
  dirLight.shadow.camera.near = 1;
  dirLight.shadow.camera.far = 1000;
  dirLight.shadow.mapSize.width = 4096;
  dirLight.shadow.mapSize.height = 4096;
  scene.add(dirLight);
  renderer.render(scene, camera);

  let dom = document.getElementById('webgl-output') as HTMLElement;
  dom.appendChild(renderer.domElement);

  function animate() {
    requestAnimationFrame(animate);
    car.position.x =
      car.position.x > (boardWidth / 2) * zoom
        ? -((boardWidth / 2) * zoom)
        : car.position.x + 1;
    renderer.render(scene, camera);
  }
  requestAnimationFrame(animate);
  var controls = new OrbitControls(camera, renderer.domElement);
  // 监听控制器的鼠标事件，执行渲染内容
  controls.addEventListener('change', () => {
    renderer.render(scene, camera);
  });
  addEventListener('click', handleBoard);
  function handleBoard(event: any) {
    chooseMesh = null; // 已被选中的模型
    let Sx = event.clientX;
    let Sy = event.clientY;
    let x = (Sx / window.innerWidth) * 2 - 1; //归一化
    let y = -(Sy / window.innerHeight) * 2 + 1;
    let raycaster = new THREE.Raycaster(); // 给模型绑定点击事件，射线，用于鼠标拾取
    raycaster.setFromCamera(new THREE.Vector2(x, y), camera); // 用一个新的原点和方向向量来更新射线（ray），X 和 Y 分量应该介于 -1 和 1 之间
    const intersects = raycaster.intersectObjects(intersectsArr, true); // 检查射线和物体之间的所有交叉点，返回按距离排序，最接近的为第一个。true还检查所有后代
    if (intersects.length > 0) {
      console.log('当前存在节点');
      dialogVisable.value = true;
      boardText.value = intersects[0].object?.parent?.name;
      console.log(intersects[0].object?.parent?.name);
    }
  }
};

const closeDialog = () => {
  dialogVisable.value = false;
};
//

onMounted(() => {
  init();
  console.log('构建三维场景');
});
</script>

<template>
  <div id="webgl-output"></div>
  <div>
    弹窗
    <a-modal
      v-model:visible="dialogVisable"
      :title="`${boardText}`"
      @ok="closeDialog"
    >
      <p>当前设备温度...</p>
      <p>当前设备状态...</p>
      <p>当前设备位置...</p>
    </a-modal>
  </div>
</template>

<style scoped>
.dialog {
  width: 100px;
  height: 100px;
  background: rgba(0, 0, 0, 0.2);
  position: absolute;
  left: 0;
  top: 0;
}
</style>
