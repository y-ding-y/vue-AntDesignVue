<template>
    <div style=" position: relative;" :style="style">
        <!-- <a-button style="z-index: 999;" type="primary" @click="handle_down">asdddddd</a-button>  -->
        <div id="myDiv">
            <video style="z-index: 999;height: 1px;width: 1px;position: absolute;top: 0;" id="video" autoplay loop muted
                ref="videsref">
                <source src="./bg.mp4">
                <!-- <source ref="videos" type="video/mp4" :src="srcs+'/file/217299'"> -->
            </video>
        </div>


        <canvas :key="cavkey" id="threes" :style="{height:ss.height+'px',width:ss.width+'px'}"></canvas>
        <!-- :style="{height: (cheight)  +'px',width:'100%'} " -->

    </div>
</template>

<script>

    import * as THREE from 'three'
    import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
    // import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
    import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
    import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
    import { GLTFExporter } from 'three/examples/jsm/exporters/GLTFExporter.js';
    import { TWEEN } from "three/examples/jsm/libs/tween.module.min.js";

    import { mapState } from 'vuex'
    import * as Mfun from './model.js'
    //import bus from './bus.js'
    const fullHeight = window.screen.height
    const fullWidth = window.screen.width
    export default {
        props: {
            cheight: { type: Number, default: 1000 },
            ss: {
                type: Object, default: () => ({
                    left: 0,
                    width: 100,
                    height: 100,
                })
            },
            vwidth: { type: Number, default: 1920 },
            vheight: { type: Number, default: 1081 },
            curdiv: { type: Boolean, default: false }
        },
        data() {
            return {
                clickfun: false,

                actionstatus: "",
                cavkey: 0,
                fullHeight,
                fullWidth,
                scene: "",

                outlinePass: "",
                c: "",
                x: 0,
                oldp: { x: 0, y: 0, z: 0 },
                oldt: { x: -8, y: 0, z: 0 },
                selectedObjects: [],
                visible: true,

                stylemodel: [],

                mainmaterial: {
                    alphaMap: null,
                    envMap: null,
                    envMapIntensity: 0.8,
                    roughness: 0.1, //粗糙度
                    metalness: 0.8,//金屬度 
                    transparent: true,
                    opacity: 1,
                },
                modelselected: [],
                progress: 0,
                result: null,
                srcs: process.env.VUE_APP_API_WEB_URL,
                refreshpublic: null,
                intervalmodel: [],
            }
        },
        created() {
            this.scene = "";
            if (this.datebind) {
                this.handle_search()
            }

            window.addEventListener('dblclick', this.onMousedblClick, false);

            window.addEventListener('click', this.onMouseClick, false);
        },
        mounted() {

        },
        watch: {
            intervalmodel() {
                this.intervalmodel.map(e => {
                    e.intervalflag = setInterval(() => {
                        this.$store.dispatch("tdpublic/refreshdata",
                            { key: Number(e.tempapi.key) }).then(() => {
                                Mfun.makeTextSprite(e.model, e.content, e.prop, e.param)
                            })



                    }, e.interval * 1000 * 60);
                })

            },
            modelstyle() {
                this.handle_changestyle();
            },
            modelselected() {
                this.handle_modelselect();
            },
            clickfun() {
                this.selectedObjects = []
                this.outlinePass.selectedObjects = [];
            },
            datebind(vnew, vold) {
                if (this.datebind && vnew != vold) {
                    this.handle_search();
                }
                else {
                    this.result = null
                }
            },
            result() {
                this.initThree()
            },
            datedetail() {
                this.handle_search();
            },
            datesource() {
                this.handle_search();
            },
        },
        beforeDestroy() {
            this.clearScene()
            clearInterval(this.refreshpublic);
            this.refreshpublic = null;
            this.intervalmodel.map(e => {
                clearInterval(e.intervalflag)
                e.intervalflag = null
            })
        },
        methods: {
            handle_down() {
                let gltfExporter = new GLTFExporter();
                // 单个模型数据 obj 
                gltfExporter.parse(this.scene.getObjectByName('world'), function (result) {
                    let link = document.createElement('a');
                    link.style.display = 'none';
                    document.body.appendChild(link);
                    link.href = URL.createObjectURL(new Blob([JSON.stringify(result)], { type: 'text/plain' }));
                    link.download = 'asd.glb';
                    link.click();
                    document.body.removeChild(link);
                });
            },


            handle_search() {
                //let dtype = this.ss.datetype;
                let dsource = this.ss.datesource 
                switch (dsource) {
                    case "publicdata":
                        this.handle_publicdata(dsource, this.ss.datetype);
                        break;
                    default:
                        break;
                }

            },
            handle_publicdata(dsource, dtype) {
                let temp = eval("(" + this.ss[dsource] + ")")
                let publicdatatab = Array(this.$localForage.getItem('3dpublicdata'))
                Promise.all(publicdatatab).then(value => {
                    if (value[0] != null) {
                        if (value[0].length > 0) {
                            if (dtype == 'string') {
                                var table = value[0][Number(temp.key)].tabledata
                                this.result = table[Number(temp.row) - 1][temp.colname]
                            } else if (dtype == "array") {
                                this.result = value[0][Number(temp.key)].tabledata
                            }
                        }
                    }
                })
            },


            handle_modelselect() {
                console.log(this.modelselected)
            },
            handle_changestyle() {
                if (this.modelstyle.length > 0) {
                    this.scene.traverse(o => {
                        this.modelstyle.map(e => {
                            if (e.name.includes(o.name)) {
                                let temp_style = eval("(" + e.style + ")")
                                let temp_model = this.scene.getObjectByName(o.name).parent.parent.parent

                                if (temp_style.binddata) {
                                    temp_model.userData.binddata = temp_style.binddata
                                    let binddata = temp_style.binddata
                                    var table = this.publicdata[Number(binddata.key)].tabledata
                                    var url = "/modelapi/uploads/" + table[Number(binddata.row) - 1][binddata.colname]
                                    let temp = this.scene.getObjectByName(o.name).parent
                                    let loader = new GLTFLoader()
                                    loader.load(url, (gltf) => {
                                        let model = gltf.scene.children[0]
                                        if (binddata.rotation && model.type == "Group") {
                                            model.rotation.x = binddata.rotation[0]
                                            model.rotation.y = binddata.rotation[1]
                                            model.rotation.z = binddata.rotation[2]
                                            // delete temp_style.rotation
                                        }
                                        let tempmaterial = new THREE.MeshStandardMaterial({ ...this.mainmaterial, ...binddata.style });
                                        if (model.type == "Group") model.children[0].material = tempmaterial
                                        else if (model.type == "Mesh") model.material = tempmaterial

                                        temp.remove(this.scene.getObjectByName(o.name))
                                        temp.add(model)
                                    })
                                    delete temp_style.binddata
                                }

                                if (temp_style.scale) {
                                    temp_model.scale.x = temp_model.scale.x * temp_style.scale[0]
                                    temp_model.scale.y = temp_model.scale.y * temp_style.scale[1]
                                    temp_model.scale.z = temp_model.scale.z * temp_style.scale[2]
                                    delete temp_style.scale
                                }

                                if (temp_style.position) {
                                    temp_model.position.x = temp_model.position.x + temp_style.position[0]
                                    temp_model.position.y = temp_model.position.y + temp_style.position[1]
                                    temp_model.position.z = temp_model.position.z + temp_style.position[2]
                                    delete temp_style.position
                                }

                                if (temp_style.moved) {
                                    Mfun.createLine(temp_style.moved, this.scene)
                                    delete temp_style.moved
                                }

                                if (temp_style.makeTextSprite) {
                                    let paras = temp_style.makeTextSprite
                                    let prop = {
                                        width: paras.width,
                                        height: paras.height,
                                        pos: paras.pos ? paras.pos : [0, 4, temp_model.position.z],
                                        scale: paras.scale,
                                    }
                                    var material = new THREE.LineBasicMaterial({ color: 0xffff00, linewidth: 100 });

                                    let lineArr = []
                                    paras.lines
                                        ? (function () {
                                            for (let i = 0; i < paras.lines.length; i += 3) {
                                                lineArr.push(new THREE.Vector3(paras.lines[i], paras.lines[i + 1], paras.lines[i + 2]))
                                            }
                                        }())
                                        : (function () {
                                            lineArr.push(new THREE.Vector3(0, 4, temp_model.position.z + 1))
                                            lineArr.push(new THREE.Vector3(0, temp_model.position.y + temp_model.scale.y * 2, temp_model.position.z))
                                        }())

                                    const curve = new THREE.CatmullRomCurve3(lineArr)
                                    const geometry = new THREE.BufferGeometry().setFromPoints(curve.getPoints(1))

                                    var line = new THREE.Line(geometry, material);
                                    temp_model.add(line);

                                    paras.content.map((tab) => {
                                        tab.map((d) => {
                                            if (typeof (d) != "string") {
                                                if (d.interval) {
                                                    this.intervalmodel.push({
                                                        interval: d.interval,
                                                        model: temp_model,
                                                        content: [paras.content],
                                                        prop: [prop],
                                                        tempapi: d,
                                                        param: {
                                                            color: 'rgba(255, 255, 255, 1)',
                                                            borderColor: 'rgba(39, 179, 236, 1)',
                                                            backgroundColor: '#57b3ed7b',
                                                            fontsize: paras.fontsize ? paras.fontsize : 3
                                                        },
                                                        intervalflag: null
                                                    })
                                                }
                                            }
                                        })
                                    })

                                    Mfun.makeTextSprite(temp_model, [paras.content], [prop], {
                                        color: 'rgba(255, 255, 255, 1)',
                                        borderColor: 'rgba(39, 179, 236, 1)',
                                        backgroundColor: '#57b3ed7b',
                                        fontsize: paras.fontsize ? paras.fontsize : 3
                                    })
                                    delete temp_style.makeTextSprite
                                }

                                if (Object.keys(temp_style).length > 0) {
                                    this.scene.getObjectByName(o.name).children.map(m => {
                                        m.material = null
                                        let tempmaterial = new THREE.MeshStandardMaterial({ ...this.mainmaterial, ...temp_style });
                                        m.material = tempmaterial
                                    })
                                }

                            }
                        })
                    })
                }


            },
            // 清空场景，包括 scene 场景下的动画，子物体，renderer,camera,control，以及自己使用过的变量置空处理 等
            clearScene() {
                cancelAnimationFrame(this.animationId);
                this.scene.traverse((child) => {
                    if (child.material) {
                        child.material.dispose();
                    }
                    if (child.geometry) {
                        child.geometry.dispose();
                    }
                    child = null;
                });

                // 场景中的参数释放清理或者置空等 
                this.renderer.forceContextLoss();
                this.renderer.dispose();
                this.scene.clear();
                this.scene = null;
                this.camera = null;
                this.controls = null;
                this.renderer.domElement = null;
                this.renderer = null;
            },
            initThree() {
                let composer
                const group = new THREE.Group();
                const scene = new THREE.Scene()

                let bgtext = new THREE.TextureLoader().load(
                    '/model/cvb1.jpg'
                )
                this.mainmaterial.alphaMap = bgtext
                var cubeTextureLoader = new THREE.CubeTextureLoader();
                cubeTextureLoader.setPath('/model/');
                var cubeTexture = cubeTextureLoader.load([
                    'bg4.png', 'bg4.png',
                    'bg4.png', 'bg4.png',
                    'bg4.png', 'bg4.png'
                ]);
                scene.environment = cubeTexture;

                this.mainmaterial.envMap = cubeTexture

                const canvas = document.getElementById('threes')
                const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true })
                renderer.setClearAlpha(0.1)
                // renderer.shadowMap.enabled = true
                this.renderer = renderer;
                const camera = new THREE.PerspectiveCamera(
                    50,
                    canvas.clientWidth / canvas.clientHeight,
                    1,
                    100
                )
                camera.position.z = 15
                camera.position.y = 19
                camera.position.x = -50

                this.camera = camera;

                scene.fog = new THREE.Fog('#000', 60, 100)
                // var axis = new THREE.AxesHelper(3000); // 3表示轴的长度  显示坐标轴
                // scene.add(axis);
                let loader = new GLTFLoader()
                //  loader.load(process.env.VUE_APP_WEB_URL+this.result,  (gltf) => {
                loader.load('/modelapi/uploads/' + this.result, (gltf) => {
                    let model = gltf.scene
                    model.name = "world"
                    //model.add(this.camera) 
                    model.traverse((o) => {
                        if (o.type == 'Group') {
                            this.stylemodel.push({ id: o.id, name: o.name, type: o.type })
                        }
                        let material1 = new THREE.MeshStandardMaterial();
                        material1.alphaMap = bgtext
                        material1.envMap = cubeTexture
                        material1.envMapIntensity = 0.8
                        material1.roughness = 0.1 //粗糙度
                        material1.metalness = 0.8 //金屬度 
                        material1.transparent = true
                        o.material = material1
                    })

                    group.add(model)
                    scene.add(group)
                    this.$localForage.setItem('3dmodel', this.stylemodel)
                })

                this.group = group;

                var video = document.querySelector("#video")
                scene.add(Mfun.videofloor(video)) // scene.add(Mfun.normalfloor())  

                //  scene.add(Mfun.normalfloor())
                const ambient = new THREE.AmbientLight("#fff", 1);
                scene.add(ambient);

                const light = new THREE.DirectionalLight(0xaaaaaa)
                light.position.set(10, 10, 10)
                scene.add(light)

                scene.add(Mfun.RectAreaLight(15, 4, 0))
                scene.add(Mfun.RectAreaLight(-15, 4, 0))

                this.scene = scene;

                setTimeout(() => {
                    this.handle_changestyle();
                }, 3000);

                this.controls = Mfun.Controls(this.camera, renderer.domElement) //控制器 
                composer = new EffectComposer(renderer);

                const renderPass = new RenderPass(this.scene, camera);
                composer.addPass(renderPass);
                this.outlinePass = Mfun.outline(canvas.offsetWidth, canvas.offsetHeight, this.scene, camera);

                composer.addPass(this.outlinePass);

                var that = this;
                function animate() {
                    if (that.controls != null && that.camera != null) {
                        that.controls.target.set(that.oldt.x, 2, that.oldt.z);
                        that.controls.update()
                        renderer.render(that.scene, that.camera)
                        composer.render(that.scene, that.camera);
                        requestAnimationFrame(animate)
                        TWEEN.update();
                        if (resizeRendererToDisplaySize(renderer)) {
                            const canvas = renderer.domElement
                            that.camera.aspect = canvas.clientWidth / canvas.clientHeight
                            that.camera.updateProjectionMatrix()
                        }
                    }

                }
                animate()

                function resizeRendererToDisplaySize(renderer) {
                    const canvas = renderer.domElement
                    var height = window.innerHeight
                    var width = window.innerWidth
                    var canvasPixelWidth = canvas.width / window.devicePixelRatio
                    var canvasPixelHeight = canvas.height / window.devicePixelRatio
                    const needResize =
                        canvasPixelWidth !== width || canvasPixelHeight !== height
                    if (needResize) {
                        composer.setSize(width, height, false)
                        renderer.setSize(width, height, false)
                    }
                    return needResize
                }

            },
            handle_getdate(temp) {
                var res = "";
                var table = this.publicdata[Number(temp.key)].tabledata
                res = table[Number(temp.row) - 1][temp.colname]

                if (temp.interval) {
                    this.refreshpublic = setInterval(() => {
                        this.$store.dispatch("tdpublic/refreshdata",
                            { key: Number(temp.key) }).then(() => {
                                res = this.$store.getters['tdpublic/commondata'][Number(temp.key)].tabledata[Number(temp.row) - 1][temp.colname]

                            })
                    }, Number(temp.interval) * 1000 * 60);
                }
                else {
                    clearInterval(this.refreshpublic);
                    this.refreshpublic = null;
                }
                return res
            },

            addSelectedObject(object) {
                this.selectedObjects = [];
                this.selectedObjects.push(object);
            },
            onMouseClick(event) {
                var res = [{ userData: { name: "", binddata: {} } }];
                if (this.clickfun) {
                    let intersects = this.getIntersects(event, this.group);
                    if (intersects.length !== 0) {
                        var selectObject = intersects[0].object;
                        selectObject = intersects[0].object.parent.parent.parent.parent;
                        if (selectObject.name.indexOf('World') == -1) {
                            this.addSelectedObject(selectObject);
                            this.outlinePass.selectedObjects = this.selectedObjects;
                        }
                    } else {
                        console.log('未选中 Mesh!');
                        this.outlinePass.selectedObjects = []
                    }
                    if (this.outlinePass.selectedObjects.length > 0) {
                        res = this.outlinePass.selectedObjects
                    }
                    return res;
                }
                else {
                    return res
                }
            },
            onMousedblClick(event) {
                var res = [{ userData: { name: "", binddata: {} } }];
                if (this.clickfun) {
                    let intersects = this.getIntersects(event, this.group);
                    if (intersects.length !== 0) {
                        var selectObject = intersects[0].object;
                        selectObject = intersects[0].object.parent.parent.parent.parent;
                        this.addSelectedObject(selectObject);
                        this.outlinePass.selectedObjects = this.selectedObjects;
                        var pos = selectObject.getWorldPosition(new THREE.Vector3())
                        this.movecamera(
                            this.oldp,
                            this.oldt,
                            { x: pos.x, y: 6, z: pos.z + 15 },
                            { x: pos.x, y: 6, z: pos.z });
                        this.oldp = { x: pos.x, y: 6, z: pos.z + 15 }
                        this.oldt = { x: pos.x, y: 6, z: pos.z }
                    } else {
                        console.log('未选中 Mesh!');
                        this.outlinePass.selectedObjects = []
                        this.movecamera(
                            this.oldp,
                            this.oldt,
                            { x: -6, y: 15, z: 35 },
                            { x: -6, y: 2, z: 0 });
                        this.oldp = { x: -6, y: 15, z: 35 }
                        this.oldt = { x: -6, y: 2, z: 0 }
                    }
                    if (this.outlinePass.selectedObjects.length > 0) {
                        res = this.outlinePass.selectedObjects
                    }
                    return res;
                }
                else {
                    return res
                }
            },

            getIntersects(event, e) {
                let rayCaster = new THREE.Raycaster();
                let mouse = new THREE.Vector2();

                const canvas = document.getElementById('threes')
                // console.log(canvas.offsetWidth, event.clientX)
                // mouse.x = ((event.clientX - canvas.getBoundingClientRect().left) / canvas.offsetWidth) * 2 - 1;
                // mouse.y = -((event.clientY - canvas.getBoundingClientRect().top) / canvas.offsetHeight) * 2 + 1; //这里为什么是-号，没有就无法点中

                let canvasBounds = canvas.getBoundingClientRect();
                mouse.x = ((event.clientX - canvasBounds.left) / (canvasBounds.right - canvasBounds.left)) * 2 - 1;
                mouse.y = - ((event.clientY - canvasBounds.top) / (canvasBounds.bottom - canvasBounds.top)) * 2 + 1;

                rayCaster.setFromCamera(mouse, this.camera);

                let intersects = rayCaster.intersectObjects(e.children, true);
                return intersects;
            },
            movecamera(oldP, oldT, newP, newT) {
                let tween = new TWEEN.Tween({
                    x1: oldP.x, y1: oldP.y, z1: oldP.z,
                    x2: oldT.x, y2: oldT.y, z2: oldT.z,
                });
                tween.to({
                    x1: newP.x, y1: newP.y, z1: newP.z,
                    x2: newT.x, y2: newT.y, z2: newT.z,
                }, 1000);
                let self = this;// 每一帧执行函数 、这个地方就是核心了、每变一帧跟新一次页面元素 
                tween.onUpdate((object) => {
                    self.camera.position.set(object.x1, object.y1, object.z1);
                    self.controls.target0.x = object.x2;
                    self.controls.target0.y = object.y2;
                    self.controls.target0.z = object.z2;
                    self.controls.update();
                });

                tween.onComplete(() => { // 动画完成后的执行函数
                    this.controls.enabled = true;
                });
                tween.easing(TWEEN.Easing.Cubic.InOut);
                tween.start();  // 这个函数必须有、这个是启动函数、不加不能启动
            }
        },
        computed: {
            publicdata() {
                return this.$store.getters['tdpublic/commondata']
            },
            ...mapState('setting', ['lang']),
            ...mapState('tdpublic', ['dept',]),
            style() {
                return {
                    position: 'absolute',
                    top: '0',
                    left: this.ss.left + 'px',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                    backgroundColor: 'antiquewhite',
                    zIndex: this.ss.zIndex
                }
            },
            modelstyle() {
                return this.ss.modelstyle
            },
            datebind() {
                return this.ss.datebind
            },
            datesource() {
                return this.ss.datesource
            },
            datedetail() {
                return this.ss[this.ss.datesource]
            },
        },

    }
</script>

<style lang="less" scoped>
    #threes {
        background-image: linear-gradient(rgb(0, 0, 0), rgb(0, 0, 0))
    }

    .v-enter-active,
    .v-leave-active {
        transition: opacity 0.8s ease;
    }

    .v-enter-from,
    .v-leave-to {
        opacity: 0;
    }
</style>

<style>
    .test {
        color: #8bffff
    }

    .seamless-warp {
        overflow: hidden;
    }
</style>
