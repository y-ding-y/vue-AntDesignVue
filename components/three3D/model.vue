<template>
    <div style=" position: relative;" :style="style">
        <video style="z-index: 999;height: 1px;width: 1px;position: absolute;top: 0;" id="video" autoplay loop muted>
            <source src="../../../public/model/科技感网格(1)(2).mp4">
        </video>
        <canvas :key="cavkey" id="threes" :style="{height:vheight+'px',width:vwidth+'px'}"></canvas>
        <!-- :style="{height: (cheight)  +'px',width:'100%'} " -->

    </div>
</template>

<script>

    import * as THREE from 'three'
    import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
    import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
    import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
    import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass.js';

    import { mapState } from 'vuex'
    //import jsons from './model.json'

    const fullHeight = window.screen.height
    const fullWidth = window.screen.width
    // const count = 20
    // const gColor = '#28f260'
    // const prop1 = {
    //     width: 800,
    //     height: 200,
    //     //  pos: [10, 5, 10],
    //     // scale:[24, 6, 0]
    //     scale: [0.24375 * count, 0.065625 * count, 0]
    // }
    // const tab1 = [
    //     ['一级高压泵后压力：', '20.2bar', gColor],
    //     ['一级循环泵后压力：', '0.2bar',]
    // ]
    // const tab2 = [
    //     ['一级高压泵后压力：', 'asd', gColor],
    //     ['一级循环泵后压力：', 'dsa',]
    // ]
    // const props = [prop1]
    // const tabs = [tab1, tab2]

    export default {
        props: {
            line: {
                type: String,
                default: 'line1',
            },
            cheight: { type: Number, default: 1000 },
            ss: {
                type: Object, default: () => ({
                    left: 0,
                    width: 100,
                    height: 100,
                })
            },
            vwidth: { type: Number, default: 1920 },
            vheight: { type: Number, default: 1081 }

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
                publicdata: [],
                progress: 0,
            }
        },
        created() {
            this.scene = "";
            let publicdatatab = Array(this.$localForage.getItem('3dpublicdata'))
            Promise.all(publicdatatab).then(value => {
                if (value[0] != null) {
                    if (value[0].length > 0) {
                        this.publicdata = value[0];
                    }
                }
            })

        },
        watch: {
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
        },
        beforeDestroy() {
            this.clearScene()
        },
        mounted() {
            this.initThree()
        },
        methods: {
            handle_modelselect() {
                console.log(this.modelselected)
            },
            handle_changestyle() {
                this.scene.traverse(o => {
                    this.modelstyle.map(e => {
                        if (e.name.includes(o.name)) {
                            let temp_style = eval("(" + e.style + ")")
                            let temp_model = this.scene.getObjectByName(o.name).parent.parent.parent


                            if (temp_style.binddata) {
                                temp_model.userData.binddata = temp_style.binddata
                                let binddata = temp_style.binddata
                                var table = this.publicdata[Number(binddata.key)].tabledata
                                var url = URL.createObjectURL(table[Number(binddata.row) - 1][binddata.colname])
                                let temp = this.scene.getObjectByName(o.name).parent
                                let loader = new GLTFLoader()
                                loader.load(url, (gltf) => {
                                    let model = gltf.scene.children[0]
                                    if (temp_style.rotation) {
                                        model.rotation.x = temp_style.rotation[0]
                                        model.rotation.y = temp_style.rotation[1]
                                        model.rotation.z = temp_style.rotation[2]
                                        delete temp_style.rotation
                                    }
                                    let tempmaterial = new THREE.MeshStandardMaterial({ ...this.mainmaterial, ...binddata.style });
                                    model.children[0].material = tempmaterial

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

                            if (temp_style.makeTextSprite) {
                                let paras = temp_style.makeTextSprite
                                let prop = {
                                    width: paras.width,
                                    height: paras.height,
                                    pos: paras.pos ? paras.pos : [0, 4, temp_model.position.z],
                                    scale: paras.scale,
                                }
                                var material = new THREE.LineBasicMaterial({ color: 0xffff00, linewidth: 10 });
                                const curve = new THREE.CatmullRomCurve3([
                                    new THREE.Vector3(0, 4, temp_model.position.z + 1),
                                    new THREE.Vector3(0, temp_model.position.y + temp_model.scale.y * 2, temp_model.position.z),
                                ])
                                const geometry = new THREE.BufferGeometry().setFromPoints(curve.getPoints(1))

                                var line = new THREE.Line(geometry, material);
                                temp_model.add(line);

                                this.makeTextSprite(temp_model, [paras.content], [prop], {
                                    color: 'rgba(255, 255, 255, 1)',
                                    borderColor: 'rgba(39, 179, 236, 1)',
                                    backgroundColor: '#57b3ed7b',
                                    fontsize: paras.fontsize ? paras.fontsize : 3
                                })
                                console.log(paras.fontsize)
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
                this.flows = [];
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
                var axis = new THREE.AxesHelper(3000); // 3表示轴的长度  显示坐标轴
                scene.add(axis);
                let loader = new GLTFLoader()
                loader.load('/model/smtonly.glb', (gltf) => {
                    let model = gltf.scene

                    //model.add(this.camera) 
                    model.traverse((o) => {
                        if (o.name == 'node_CombineFloor_-197068') {
                            o.parent.remove(o)
                        }
                        // if (o.type == 'Group') {
                        //     console.log(o.name)
                        //     this.stylemodel.push({ id: o.id, name: o.name, type: o.type })
                        // }
                        let material1 = new THREE.MeshStandardMaterial();
                        material1.alphaMap = bgtext
                        //   material1.envMap = envMap
                        material1.envMap = cubeTexture
                        material1.envMapIntensity = 0.8
                        material1.roughness = 0.1 //粗糙度
                        material1.metalness = 0.8 //金屬度 
                        material1.transparent = true
                        o.material = material1
                        // console.log(o.name)
                        // o.children[0].children[0].children[0].children 
                    })

                    group.add(model)
                    scene.add(group)
                    //this.$localForage.setItem('3dmodel', this.stylemodel)
                })

                loader.load('/model/cart.glb', (gltf) => {
                    let model = gltf.scene.children[0]
                    model.name = "cart";
                    model.rotation.y = 0.5 * Math.PI
                    group.add(model)
                    scene.add(group)
                })


                var video = document.querySelector("#video")
                var texture = new THREE.VideoTexture(video);

                texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
                texture.repeat.set(5, 10);
                var material = new THREE.MeshLambertMaterial({
                    map: texture,
                });
                //创建普通的平面几何体
                var gemotery = new THREE.PlaneGeometry(100, 100);
                //创建网格对象
                var mesh = new THREE.Mesh(gemotery, material);
                mesh.position.y = -0.001;
                mesh.rotation.x = -0.5 * Math.PI
                scene.add(mesh);

                this.group = group;
                const ambient = new THREE.AmbientLight("#fff", 1);
                scene.add(ambient); //将环境光添加到场景中 

                const light = new THREE.DirectionalLight(0xaaaaaa)
                //light.position.set(0, -15, 10)
                light.position.set(10, 10, 10)
                scene.add(light)

                let rectLight2 = new THREE.RectAreaLight(0xffffff, 5, 30, 60);
                rectLight2.position.set(15, 4, 0);
                rectLight2.rotation.x = -0.5 * Math.PI;
                scene.add(rectLight2);

                let rectLight = new THREE.RectAreaLight(0xffffff, 5, 30, 60);
                rectLight.position.set(-15, 4, 0);
                rectLight.rotation.x = -0.5 * Math.PI;
                scene.add(rectLight);


                this.scene = scene;

                setTimeout(() => {
                    this.handle_changestyle();
                    this.createLine();
                }, 3000);
                this.controls = new OrbitControls(this.camera, renderer.domElement)
                this.controls.enableDamping = false//阻尼感
                this.dampingFactor = 0.25
                this.controls.enablePan = true
                this.controls.minPolarAngle = 0;
                this.controls.maxPolarAngle = 0.49 * Math.PI;
                // this.controls.minZoom = 0;
                this.controls.minDistance = 1;
                //	this.controls.maxDistance = 100;  

                composer = new EffectComposer(renderer);

                const renderPass = new RenderPass(this.scene, camera);
                composer.addPass(renderPass);


                const outlinePass = new OutlinePass(
                    new THREE.Vector2(
                        canvas.offsetWidth, canvas.offsetHeight),
                    this.scene, camera);

                outlinePass.visibleEdgeColor.set("green"); // 边缘可见部分发颜色 sColor[0].color
                outlinePass.hiddenEdgeColor.set("orange"); // 边缘遮挡部分发光颜色 sColor[1].color
                outlinePass.edgeStrength = Number(10.0); //边框的亮度
                outlinePass.edgeGlow = Number(0.6); //光晕[0,1]
                outlinePass.edgeThickness = Number(1.0); //边缘浓度
                outlinePass.pulsePeriod = Number(1.8); //呼吸闪烁的速度 闪烁频率 ，默认0 ，值越大频率越低
                outlinePass.usePatternTexture = false; //是否使用父级的材质
                outlinePass.downSampleRatio = 2; // 边框弯曲度
                outlinePass.clear = true;

                this.outlinePass = outlinePass;

                composer.addPass(outlinePass);


                this.controls.target.set(-8, 0, 0);
                var that = this;
                function animate() {
                    if (that.controls != null) that.controls.update()
                    renderer.render(that.scene, that.camera)
                    composer.render(that.scene, that.camera);
                    requestAnimationFrame(animate)
                    if (resizeRendererToDisplaySize(renderer)) {
                        const canvas = renderer.domElement
                        that.camera.aspect = canvas.clientWidth / canvas.clientHeight
                        that.camera.updateProjectionMatrix()
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
                console.log(res)
                return res
                // return res;
            },
            makeTextSprite(scene, tabs, props, parameters) {
                let m = scene.getObjectByName(scene.name + "sprite")
                if (m) m.parent.remove(m)
                if (parameters === undefined) parameters = {}
                tabs.forEach((tab, k) => {
                    let {
                        width,
                        height
                    } = props[k]
                    /* 创建画布 */
                    let canvas = document.createElement('canvas');
                    let context = canvas.getContext('2d')
                    canvas.width = width
                    canvas.height = height
                    let gap = 10

                    let fontface = Object.prototype.hasOwnProperty.call(parameters, 'fontface') ? parameters["fontface"] : "sans-serif"
                    /* 字体大小 */
                    let fontsize = Object.prototype.hasOwnProperty.call(parameters, 'fontsize') ?
                        parameters["fontsize"] * 16 : 30
                    let color = Object.prototype.hasOwnProperty.call(parameters, 'color') ? parameters["color"] : 'rgba(0, 0, 0, 1.0)'
                    /* 边框厚度 */
                    let borderWith = Object.prototype.hasOwnProperty.call(parameters, 'borderWith') ? parameters["borderWith"] : 2
                    /* 边框颜色 */
                    let borderColor = Object.prototype.hasOwnProperty.call(parameters, 'borderColor') ? parameters["borderColor"] : {
                        r: 0,
                        g: 0,
                        b: 0,
                        a: 1.0
                    }
                    /* 背景颜色 */
                    let backgroundColor = Object.prototype.hasOwnProperty.call(parameters, 'backgroundColor') ? parameters["backgroundColor"] : {
                        r: 255,
                        g: 255,
                        b: 255,
                        a: 1.0
                    }

                    /* 字体加粗 */
                    context.font = "Bold " + fontsize + "px " + fontface
                    let unit = gap + fontsize
                    /* 背景颜色 */
                    context.fillStyle = backgroundColor
                    /* 边框的颜色 */
                    context.strokeStyle = borderColor
                    context.lineWidth = borderWith
                    /* 绘制圆角矩形 */
                    this.roundRect(context, gap, gap, width - gap, height - gap, 4)

                    tab.forEach((d, i) => {
                        d = d.map(e => {
                            if (typeof (e) != "string") {
                                return this.handle_getdate(e)
                            }
                            else {
                                return e
                            }
                        })
                        context.fillStyle = color;
                        context.fillText(d[0], gap * 2, gap + unit * (i + 1))
                        if (d[2]) {
                            context.fillStyle = d[2]
                        }
                        context.fillText(d[1], gap * 2 + this.measureText(d[0], context), gap + unit * (i + 1))
                    })
                    // this.circle(context, 350, 180, width - gap, 20, 20, "red");
                    // this.circle(context, 300, 180, width - gap, 20, 20, "green");

                    let texture = new THREE.Texture(canvas);
                    texture.needsUpdate = true
                    let spriteMaterial = new THREE.SpriteMaterial({
                        map: texture,
                    });
                    let sprite = new THREE.Sprite(spriteMaterial)
                    sprite.scale.set(...props[k].scale)
                    sprite.name = scene.name + "sprite"

                    sprite.center = new THREE.Vector2(0, 0);
                    sprite.position.set(...props[k].pos);
                    scene.add(sprite);
                })
            },

            measureText(text, ctx, font) {
                if (font) ctx.font = font
                return ctx.measureText(text).width;
            },
            /* 绘制圆角矩形 */

            circle(ctx, x, y, w, h, r, color) {
                ctx.beginPath();
                ctx.arc(x, y, r, 0, Math.PI * 2, false);
                ctx.fillStyle = color;
                ctx.fill()
            },

            roundRect(ctx, x, y, w, h, r) {
                ctx.beginPath();
                ctx.moveTo(x + r, y);
                ctx.lineTo(x + w - r, y);
                ctx.quadraticCurveTo(x + w, y, x + w, y + r);
                ctx.lineTo(x + w, y + h - r);
                ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h);
                ctx.lineTo(x + r, y + h);
                ctx.quadraticCurveTo(x, y + h, x, y + h - r);
                ctx.lineTo(x, y + r);
                ctx.quadraticCurveTo(x, y, x + r, y);
                ctx.closePath();

                // ctx.shadowColor = "#qb95cf";
                // ctx.shadowOffsetX = 0;
                // ctx.shadowOffsetY = 0;
                // ctx.shadowBlur = 4;

                ctx.fill();
                ctx.stroke();
                ctx.shadowColor = "";
                ctx.shadowOffsetX = 0;
                ctx.shadowOffsetY = 0;
                ctx.shadowBlur = 0;
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
                        console.log(selectObject.name)
                        this.addSelectedObject(selectObject);
                        this.outlinePass.selectedObjects = this.selectedObjects;

                    } else {
                        console.log('未选中 Mesh!');
                    }
                    if (this.outlinePass.selectedObjects.length > 0) {
                        res = this.outlinePass.selectedObjects
                    }
                    return res;
                }
                else {
                    return res
                }
                // this.$emit('comclick', { name: "model", val: this.outlinePass.selectedObjects })
            },

            getIntersects(event, e) {
                let rayCaster = new THREE.Raycaster();
                let mouse = new THREE.Vector2();

                const canvas = document.getElementById('threes')
                mouse.x = ((event.clientX - canvas.getBoundingClientRect().left) / canvas.offsetWidth) * 2 - 1;
                mouse.y = -((event.clientY - canvas.getBoundingClientRect().top) / canvas.offsetHeight) * 2 + 1; //这里为什么是-号，没有就无法点中

                rayCaster.setFromCamera(mouse, this.camera);

                let intersects = rayCaster.intersectObjects(e.children, true);
                return intersects;
            },
            createLine() {
                //创建样条曲线，作为运动轨迹
                this.curve = new THREE.CatmullRomCurve3([
                    new THREE.Vector3(-24, 0, 5),
                    new THREE.Vector3(11, 0, 3),
                    new THREE.Vector3(11, 0, 1),
                    new THREE.Vector3(-23.5, 0, -2),
                    new THREE.Vector3(-23.5, 0, -4),
                    new THREE.Vector3(8.5, 0, -6),
                    new THREE.Vector3(8.5, 0, -7.5),
                    new THREE.Vector3(-23.5, 0, -7),
                    new THREE.Vector3(-23.5, 0, -1),
                    new THREE.Vector3(-23.5, 0, 4),
                ])
                // const geometry = new THREE.BufferGeometry().setFromPoints(this.curve.getPoints(5000))
                // // 材质对象
                // var material = new THREE.LineBasicMaterial({
                //     color: 'red'
                // })
                // // 线条模型对象
                // var line = new THREE.Line(geometry, material)
                // this.scene.add(line) // 线条对象添加到场景中
                this.render();
            },

            render() {
                requestAnimationFrame(this.render)
                if (this.progress <= 1 - 0.003) {
                    const point = this.curve.getPointAt(this.progress) //获取样条曲线指定点坐标，作为相机的位置
                    const pointBox = this.curve.getPointAt(this.progress + 0.003) //获取样条曲线指定点坐标
                    const model = this.scene.getObjectByName("cart");
                    model.position.set(point.x, point.y, point.z)
                    model.lookAt(pointBox.x, pointBox.y, pointBox.z)
                    // this.controls.position0.set(point.x, point.y + 3, point.z) //非必要，场景有控件时才加上
                    //        this.controls.target.set(pointBox.x, pointBox.y + 3, pointBox.z) //非必要，场景有控件时才加上
                    this.progress += 0.003
                } else {
                    this.progress = 0
                }
                // this.renderer.render(this.scene, this.camera)
            }
        },
        computed: {
            ...mapState('setting', ['lang']),
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
        },

    }
</script>

<style lang="less" scoped>
    #threes {
        /* background-image: linear-gradient(rgb(255, 255, 255), rgb(246, 168, 96)) */
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

    .success,
    .success:hover,
    .success:visited {
        background-color: @success-color;
        color: white;
    }

    .divclas {
        overflow-y: scroll;
        height: 200px;

    }

    .label {
        width: 100px;
        height: 100px;

    }

    .maincolor {
        color: @primary-color;
    }

    .successcolor {
        color: @success-color
    }

    .manpower {
        background-size: 120px auto;
        background-repeat: no-repeat;
        background-position: 80% 36%;
        color: @primary-3
    }
</style>

<style>
    .leftd {
        height: 80%;
        position: absolute;
        left: 1%;
        bottom: 10%;
        width: 20%;
        background-color: rgba(153, 152, 160, 0.521);
        display: flex;
        flex-direction: column;
        justify-content: space-around;
    }

    .rightd {
        position: absolute;
        right: 0;
        top: 0;
        width: 20%;
        background-color: rgba(153, 152, 160, 0.521);
        display: flex;
        flex-direction: column;
        justify-content: space-around;
    }

    .centerd {
        position: absolute;
        width: 80%;
        background-color: rgba(153, 152, 160, 0.521);
        display: flex;

    }

    .test {
        color: #e69539
    }

    .seamless-warp {
        overflow: hidden;
    }
</style>
