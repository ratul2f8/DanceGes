<script>
  import * as THREE from "three";
  import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import Loading from "./Loading.svelte";
  export let currentAnimationFBX;
  class LoadModelDemo {
    constructor() {
      this._Initialize();
    }
    _Initialize() {
      this.previousFbx = null;
      this._threejs = new THREE.WebGLRenderer({
        antialias: true,
      });
      this._threejs.shadowMap.enabled = true;
      this._threejs.shadowMap.type = THREE.PCFSoftShadowMap;
      this._threejs.setPixelRatio(window.devicePixelRatio);
      this._threejs.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(this._threejs.domElement);
      window.addEventListener(
        "resize",
        () => {
          this._OnWindowResize();
        },
        false
      );
      const fov = 20;
      const aspect = window.innerWidth / window.innerHeight;
      const near = 0.1;
      const far = 1000.0;
      this._camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
      this._camera.position.set(-8, 85, 220);
      this._scene = new THREE.Scene({
        background:
          "linear-gradient(90deg, rgba(2,0,36,1) 0%, rgba(121,9,53,1) 35%, rgba(255,0,139,1) 100%)",
      });
      let light = new THREE.DirectionalLight(0xffffff, 1.0);
      light.position.set(20, 100, 10);
      light.target.position.set(0, 0, 0);
      light.castShadow = true;
      light.shadow.bias = -0.001;
      light.shadow.mapSize.width = 2048;
      light.shadow.mapSize.height = 2048;
      light.shadow.camera.near = 0.1;
      light.shadow.camera.far = 500.0;
      light.shadow.camera.near = 0.5;
      light.shadow.camera.far = 500.0;
      light.shadow.camera.left = 100;
      light.shadow.camera.right = -100;
      light.shadow.camera.top = 100;
      light.shadow.camera.bottom = -100;
      this._scene.add(light);

      light = new THREE.AmbientLight(0xffffff, 4.0);
      this._scene.add(light);
      const controls = new OrbitControls(
        this._camera,
        this._threejs.domElement
      );
      controls.target.set(0, 20, 0);
      controls.update();
      //backeground
      const loader = new THREE.CubeTextureLoader();
      const texture = loader.load([
        "bg-2.jpg",
        "bg-2.jpg",
        "bg-2.jpg",
        "bg-2.jpg",
        "bg-2.jpg",
        "bg-2.jpg",
      ]);
      this._scene.background = texture;
      //pane
      const textureLoader = new THREE.TextureLoader();
      const paneTexture = textureLoader.load("bg.jpg");
      const plane = new THREE.Mesh(
        new THREE.PlaneGeometry(200, 200, 30, 30),
        new THREE.MeshBasicMaterial({ map: paneTexture })
      );
      plane.castShadow = false;
      plane.receiveShadow = true;
      plane.rotation.x = -Math.PI / 2;
      this._scene.add(plane);
      this._mixers = [];
      this._previousRAF = null;
      this._LoadAnimatedModel();
      this._RAF();
    }
    _LoadAnimatedModel(toLoad = "Samba-Dancing.fbx") {
      const loader = new FBXLoader();
      loader.setPath("./resources/boss/");
      loader.load("theboss.fbx", (fbx) => {
        //remove previous object
        if (this.previousFbx) {
          let selectedObject = this._scene.getObjectById(this.previousFbx);
          this._scene.remove(selectedObject);
        }
        fbx.scale.setScalar(0.25);
        fbx.traverse((c) => {
          c.castShadow = true;
        });

        //load animation
        // if (currentAnimationFBX) {
        //   let anim = new FBXLoader();
        //   anim.setPath("./resources/boss/");
        //   anim.load(toLoad, (anim) => {
        //     const m = new THREE.AnimationMixer(fbx);
        //     this._mixers.push(m);
        //     const idle = m.clipAction(anim.animations[0]);
        //     idle.play();
        //   });
        // }
        this.previousFbx = fbx.id;
        this._scene.add(fbx);
      });
    }
    _UpdateAnimation(toLoad) {
      //delete previous animation
      this._mixers = [];
      if (toLoad !== null) {
        let anim = new FBXLoader();
        anim.setPath("./resources/boss/");
        anim.load(toLoad, (anim) => {
          const m = new THREE.AnimationMixer(
            this._scene.getObjectById(this.previousFbx)
          );
          this._mixers.push(m);
          const idle = m.clipAction(anim.animations[0]);
          idle.play();
        });
      }
    }
    _OnWindowResize() {
      this._camera.aspect = window.innerWidth / window.innerHeight;
      this._camera.updateProjectionMatrix();
      this._threejs.setSize(window.innerWidth, window.innerHeight);
    }
    _RAF() {
      requestAnimationFrame((t) => {
        if (this._previousRAF === null) {
          this._previousRAF = t;
        }

        this._RAF();
        this._threejs.render(this._scene, this._camera);
        this._Step(t - this._previousRAF);
        this._previousRAF = t;
      });
    }
    _Step(timeElapsed) {
      const timeElapsedS = timeElapsed * 0.001;
      if (this._mixers) {
        this._mixers.map((m) => m.update(timeElapsedS));
      }
      if (this._controls) {
        this._controls.Update(timeElapsedS);
      }
    }
  }

  let _APP = new LoadModelDemo();
  $: sign = currentAnimationFBX.map((el) => _APP._UpdateAnimation(el));

  // window.addEventListener("DOMContentLoaded", () => {
  //   _APP = new LoadModelDemo();
  // });
</script>
