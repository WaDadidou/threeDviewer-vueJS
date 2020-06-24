<template>
    <!--    Only if WebGL is supported :-->
    <div v-if="isWebglSupported" id="viewer">
        <!--        Button to open or close GUI menu-->
        <div :class="[isFullscreen ? 'btnTguiFullscreen' : ''] + [isGuiShown ? ' guiShown' : ' '] + ' btnToggleGui'" @click="toggleGui">
            {{isGuiShown ? 'close controls' : 'open controls'}}
        </div>
        <!--        GUI : We can create a Vue component for that-->
        <ul :class="[isFullscreen && 'guiFullscreen', isGuiShown && 'shown']" class="gui">
            <li class="guiItem">
                <div class="labelInput">
                    <label for="inputModelVisible"> Model visibility </label>
                    <input v-model="modelVisible" id="inputModelVisible" type="checkbox"/>
                </div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label class="labelBold" for="inputFog"> Fog </label>
                    <input v-model="fogEnabled" id="inputFog" type="checkbox"/>
                </div>
                <button @click="toggleSubmenu" class="btnToggleSubmenu"> Fog</button>
                <!--                Submenu Fog  -->
                <ul class="submenu">
                    <li>
                        <div class="labelInput">
                            <label for="inputFogDensity"> Density </label>
                            <input ref="refInputFogDensity" v-model="fogDensity" min="0.00002" max="0.02"
                                   id="inputFogDensity" type="range"
                                   step="0.00001"/>
                        </div>
                    </li>
                </ul>
                <div class="endSubmenu"></div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label for="inputWireframe"> Wireframe </label>
                    <input v-model="wireframeEnabled" id="inputWireframe" type="checkbox"/>
                </div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label for="inputFilm"> Film </label>
                    <input v-model="filmPassEnabled" id="inputFilm" type="checkbox"/>
                </div>
            </li>
            <li v-if="format === 'glTF'" class="guiItem">
                <div class="labelInput">
                    <span class="labelBold"> Anisotropy </span>
                    <span>{{ anisoLevel}}</span>
                </div>
                <button @click="toggleSubmenu" class="btnToggleSubmenu"> Anisotropy</button>
                <!--                Submenu Anisotropy  -->
                <ul class="submenu">
                    <li>
                        <div class="labelInput">
                            <span> Max for your GPU : </span>
                            <span>{{ maxAnisotropy}}</span>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputAnisoLevel"> Samples </label>
                            <select v-model="anisoLevel" id="inputAnisoLevel">
                                <option selected="selected" :value="1">1</option>
                                <option :disabled="maxAnisotropy < 2" :value="2">2</option>
                                <option :disabled="maxAnisotropy < 4" :value="4">4</option>
                                <option :disabled="maxAnisotropy < 8" :value="8">8</option>
                                <option :disabled="maxAnisotropy < 16" :value="16">16</option>
                                <option :disabled="maxAnisotropy < 32" :value="32">32</option>
                                <option :disabled="maxAnisotropy < 64" :value="64">64</option>
                            </select>
                        </div>
                    </li>
                </ul>
                <div class="endSubmenu"></div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label for="inputSSAA"> SSAA </label>
                    <input v-model="ssaaRenderPassEnabled" id="inputSSAA" type="checkbox"/>
                </div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label for="inputRotation"> Rotate </label>
                    <input v-model="rotationEnbaled" id="inputRotation" type="checkbox"/>
                </div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label for="inputBackground"> background </label>
                    <input id="inputBackground" v-model="backgroundEnabled" type="checkbox"/>
                </div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label class="labelBold" for="inputSAO"> SAO </label>
                    <input v-model="saoPassEnabled" id="inputSAO" type="checkbox"/>
                </div>
                <button @click="toggleSubmenu" class="btnToggleSubmenu"> SAO</button>
                <!--                Submenu SAO  -->
                <ul class="submenu">
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOoutput"> output </label>
                            <select v-model="saoPassParams.output" id="inputSAOoutput">
                                <option :value="0">Beauty</option>
                                <option :value="1">Beauty+SAO</option>
                                <option :value="2">SAO</option>
                                <option :value="3">Depth</option>
                                <option :value="4">Normal</option>
                            </select>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAObias"> bias </label>
                            <input v-model="saoPassParams.saoBias" min="-1" max="1" id="inputSAObias" type="range"
                                   step="0.01"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOintensity"> intensity </label>
                            <input v-model="saoPassParams.saoIntensity" min="0" max="1" id="inputSAOintensity"
                                   type="range"
                                   step="0.01"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOscale"> scale </label>
                            <input v-model="saoPassParams.saoScale" min="0" max="10" id="inputSAOscale" type="range"
                                   step="0.1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOkRadius"> kernel radius </label>
                            <input v-model="saoPassParams.saoKernelRadius" min="1" max="100" id="inputSAOkRadius"
                                   type="range" step="1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOminResolution"> min. resolution </label>
                            <input v-model="saoPassParams.saoMinResolution" min="0" max="1" id="inputSAOminResolution"
                                   type="range" step="0.1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOblur"> blur </label>
                            <input v-model="saoPassParams.saoBlur" id="inputSAOblur" type="checkbox"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOblurRadius"> blur radius </label>
                            <input v-model="saoPassParams.saoBlurRadius" min="0" max="200" id="inputSAOblurRadius"
                                   type="range" step="0.1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOblurStdDev"> blur stdDev </label>
                            <input v-model="saoPassParams.saoBlurStdDev" min="0.5" max="150" id="inputSAOblurStdDev"
                                   type="range" step="0.1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputSAOblurDepthCutoff"> blur depth cutoff </label>
                            <input v-model="saoPassParams.saoBlurDepthCutoff" min="0" max="0.1"
                                   id="inputSAOblurDepthCutoff"
                                   type="range" step="0.001"/>
                        </div>
                    </li>
                </ul>
                <div class="endSubmenu"></div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label class="labelBold" for="inputDlight"> D Light </label>
                    <input v-model="dLightVisible" id="inputDlight" type="checkbox"/>
                </div>
                <!--                Submenu D light  -->
                <button @click="toggleSubmenu" class="btnToggleSubmenu"> D Light</button>
                <ul class="submenu">
                    <li>
                        <div class="labelInput">
                            <label for="inputDlightX"> D Light X </label>
                            <input v-model="dLightPosition.x" min="-200" max="200" id="inputDlightX"
                                   type="range" step="0.1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputDlightY"> D Light Y </label>
                            <input v-model="dLightPosition.y" min="-200" max="200" id="inputDlightY"
                                   type="range" step="0.1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputDlightZ"> D Light Z </label>
                            <input v-model="dLightPosition.z" min="-200" max="200" id="inputDlightZ"
                                   type="range" step="0.1"/>
                        </div>
                    </li>
                    <li>
                        <div class="labelInput">
                            <label for="inputDlightHelper"> Helper </label>
                            <input v-model="dLightHelperVisible" id="inputDlightHelper" type="checkbox"/>
                        </div>
                    </li>
                </ul>
                <div class="endSubmenu"></div>
            </li>
            <li class="guiItem">
                <div class="labelInput">
                    <label class="labelBold" for="inputHlight"> H Light </label>
                    <input v-model="hLightVisible" id="inputHlight" type="checkbox"/>
                </div>
                <!--                Submenu H light  -->
                <button @click="toggleSubmenu" class="btnToggleSubmenu"> H Light</button>
                <ul class="submenu">
                    <li>
                        <div class="labelInput">
                            <label for="inputHlightHelper"> Helper </label>
                            <input v-model="hLightHelperVisible" id="inputHlightHelper" type="checkbox"/>
                        </div>
                    </li>
                </ul>
                <div class="endSubmenu"></div>
            </li>
        </ul>
        <!--        Button to toggle fullscreen -->
        <div :class="[isFullscreen && 'btnFsFullscreen']" class="btnFs" @click="toggleFullscreen"> [ ]</div>
        <!--    Canvas to display the scene -->
        <canvas :class="[isFullscreen && 'canvasFullscreen']" ref="refCanvas" id="rendererDomElement"></canvas>
    </div>
    <!--    If WebGL is not supported :-->
    <div v-else>
        <slot>
            Your browser does not seem to support <a href="http://khronos.org/webgl/wiki/Getting_a_WebGL_Implementation"
                                                     style="color:#000">WebGL</a>.<br/>'
        </slot>
    </div>
</template>

<script>
import * as THREE from 'three';
import Stats from 'three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader';
import {STLLoader} from 'three/examples/jsm/loaders/STLLoader';
import {RenderPass} from "three/examples/jsm/postprocessing/RenderPass";
import {SAOPass} from "three/examples/jsm/postprocessing/SAOPass";
import {FilmPass} from "three/examples/jsm/postprocessing/FilmPass";
import {SSAARenderPass} from "three/examples/jsm/postprocessing/SSAARenderPass";
import {EffectComposer} from "three/examples/jsm/postprocessing/EffectComposer";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";

export default {
    name: "ThreeDviewer",
    props: {
        threeDFilePath: String,
        format: String
    },
    data() {
        return {
            //--- Canvas
            isFullscreen: false,
            canvas: null,
            //--- Renderer
            renderer: null,
            rendererParams: {
                clearColor: 0.0500,
                shadowMapEnabled: true,
                shadowMaptype: THREE.PCFSoftShadowMap,
                toneMappingExposure: 1,
                toneMapping: THREE.Uncharted2ToneMapping,
                gammaOutput: true,
                gammaInput: true,
                gammaFactor: 2.2
            },
            //--- Camera - Controls
            camera: null,
            cameraFar: 1000,
            cameraNear: 0.1,
            cameraFov: 75,
            cameraPosition: {x: 0, y: 10, z: 20},
            controls: null,
            controlsTarget: {x: 0, y: 5, z: 0},
            //--- Scene
            scene: null,
            sceneName: '3DViewerScene',
            maxAnisotropy: 0,
            anisoLevel: 1,
            cubeTextureLoader: null,
            backgroundEnabled: true,
            bkgrndSceneColorHex: 0x52704C,
            backgrounds: [],
            fog: null,
            fogEnabled: false,
            fogColorHex: 0xE5DAD0,
            fogDensity: 0.0050,
            //--- Lights
            hLight: null,
            hLightVisible: true,
            hLightPosition: {x: 0, y: 1, z: 0},
            hLightIntensity: 0.8,
            groundColorHex: 0x808080,
            skyColorHex: 0xaaaaff,
            dLight: null,
            dLightIntensity: 0.8,
            dLightColorHex: 0xffffff,
            dLightVisible: true,
            dLightPosition: {x: 1, y: 1, z: 1},
            dLightHelper: null,
            dLightHelperVisible: false,
            dLightHelperColorHex: 0xFF0000,
            dLightHelperSize: 50,
            hLightHelper: null,
            hLightHelperVisible: false,
            hLightHelperColorHex: 0xFF0000,
            hLightHelperSize: 50,
            //--- Models loaders
            phongMaterialColorHex: 0x888888,
            modelVisible: true,
            rotationEnbaled: false,
            rotationStep: 0.0500,
            gltfLoader: null,
            stlLoader: null,
            model: null,
            wireframe: null,
            wireframeEnabled: false,
            wireframeLines: null,
            wireframeLinesParams: {
                depthTest: false,
                opacity: 0.25,
                transparent: true
            },
            //--- Post processing
            composer: null,
            renderPass: null,
            shaderPass: null,
            shaderPassEnabled: false,
            shaderPassParams: {},
            saoPass: null,
            saoPassEnabled: false,
            saoPassDepthTexture: true,
            saoPassUseNormals: true,
            saoPassParams: {
                output: 1,
                saoBias: 0,
                saoKernelRadius: 16,
                saoMinResolution: 0.5,
                saoIntensity: 0.5,
                saoScale: 5,
                saoBlur: false,
                saoBlurRadius: 100,
                saoBlurStdDev: 75,
                saoBlurDepthCutoff: 0.05
            },
            ssaaRenderPass: null,
            ssaaRenderPassEnabled: false,
            ssaaClearColor: 0x000000,
            ssaaClearAlpha: 0,
            filmPass: null,
            filmPassEnabled: false,
            filmPassParams: {
                noiseIntensity: 0.35,
                scanlinesIntensity: 0.025,
                scanlinesCount: 648,
                grayscale: 0
            },
            //--- GUI
            stats: null,
            isGuiShown: false
        }
    },
    mounted() {
        this.init();
        this.display();
    },
    updated() {
        // Only DOM -> Using the VueJS's updated lifecycle method
        this.updateStats();
    },
    destroyed() {
        // this.cleanRenderer()
    },
    methods: {
        // ============ MAIN =========================
        init() {
            //--- Canvas
            this.canvas = this.$refs.refCanvas
            // --- Renderer
            this.renderer = new THREE.WebGLRenderer({canvas: this.canvas, alpha: true})
            this.setRenderer()
            // --- Camera and controls
            this.camera = new THREE.PerspectiveCamera(this.cameraFov, this.canvas.clientWidth / this.canvas.clientHeight, this.cameraNear, this.cameraFar)
            this.controls = new OrbitControls(this.camera, this.renderer.domElement)
            this.setCamera()
            this.setControls()
            // --- Scene
            this.scene = new THREE.Scene()
            this.cubeTextureLoader = new THREE.CubeTextureLoader();
            this.fog = new THREE.FogExp2(this.fogColorHex, this.fogDensity)
            this.setScene()
            //--- Models, textures, loaders
            this.maxAnisotropy = this.renderer.capabilities.getMaxAnisotropy();
            this.gltfLoader = new GLTFLoader()
            this.stlLoader = new STLLoader()
            if (this.format === 'STL') this.loadStl()
            else if (this.format === 'glTF') this.loadGltf();
            // --- Lights
            this.hLight = new THREE.HemisphereLight(
                    new THREE.Color(this.skyColorHex),
                    new THREE.Color(this.groundColorHex),
                    this.hLightIntensity
            )
            this.dLight = new THREE.DirectionalLight(
                    new THREE.Color(this.dLightColorHex),
                    this.dLightIntensity
            )
            this.dLightHelper = new THREE.DirectionalLightHelper(this.dLight, this.dLightHelperSize, this.dLightHelperColorHex);
            this.hLightHelper = new THREE.HemisphereLightHelper(this.hLight, this.hLightHelperSize, this.hLightHelperColorHex);
            this.setLights();
            //--- GUI
            this.stats = new Stats()
            this.setStats()
            //--- Post processing
            this.composer = new EffectComposer(this.renderer)
            this.renderPass = new RenderPass(this.scene, this.camera)
            this.saoPass = new SAOPass(this.scene, this.camera, this.saoPassDepthTexture, this.saoPassUseNormals)
            this.ssaaRenderPass = new SSAARenderPass(this.scene, this.camera, this.ssaaClearColor, this.ssaaClearAlpha);
            this.filmPass = new FilmPass(
                    this.filmPassParams.noiseIntensity,
                    this.filmPassParams.scanlinesIntensity,
                    this.filmPassParams.scanlinesCount,
                    this.filmPassParams.grayscale
            );
            this.setPostProcessing()
        },
        display() {
            requestAnimationFrame(this.display);
            this.stats.begin()
            this.render()

            // For each frame -> Updating the stuff
            // We must use this way instead of VueJS's updated lifecycle method to avoid framerate issues
            this.updateScene();
            this.updateModel();
            this.updateLight()
            this.updatePostProcessing()

            this.stats.end()
        },
        render() {
            if (this.isResizeNeeded(this.renderer)) {
                this.camera.aspect = this.canvas.clientWidth / this.canvas.clientHeight;
                this.camera.updateProjectionMatrix();
                this.composer.setSize(this.canvas.clientWidth, this.canvas.clientHeight);
            }
            this.composer.render();
        },
        // ============ MODEL FROM THE API ===========
        loadGltf() {
            this.gltfLoader.load(this.threeDFilePath, (gltf) => {
                const gltfModel = gltf.scene
                const mesh = gltfModel.children[0]
                const material = mesh.material

                // --- First setting of anisotropy
                material.map.anisotropy = this.anisoLevel
                this.model = new THREE.Mesh(mesh.geometry, material)

                // --- We init and add Wireframe here because it depends of the mesh from gltfLoader
                this.wireframe = new THREE.WireframeGeometry(mesh.geometry);
                this.wireframeLines = new THREE.LineSegments(this.wireframe);
                this.wireframeLines.material.depthTest = this.wireframeLinesParams.depthTest;
                this.wireframeLines.material.opacity = this.wireframeLinesParams.opacity
                this.wireframeLines.material.transparent = this.wireframeLinesParams.transparent

                this.scene.add(this.model);
                this.setModelControls();
            });
        },
        loadStl() {
            this.stlLoader.load(this.threeDFilePath, (geometry) => {
                const material = new THREE.MeshPhongMaterial()
                material.color = new THREE.Color(this.phongMaterialColorHex)
                this.model = new THREE.Mesh(geometry, material)

                // We init and add Wireframe here because it depends of the mesh from stlLoader
                this.wireframe = new THREE.WireframeGeometry(geometry);
                this.wireframeLines = new THREE.LineSegments(this.wireframe);
                this.wireframeLines.material.depthTest = this.wireframeLinesParams.depthTest;
                this.wireframeLines.material.opacity = this.wireframeLinesParams.opacity
                this.wireframeLines.material.transparent = this.wireframeLinesParams.transparent

                this.scene.add(this.model);
                this.setModelControls();
            });
        },
        // ============ SETTERS ======================
        setRenderer() {
            this.renderer.setClearColor(this.rendererParams.clearColor);
            this.renderer.setPixelRatio(this.canvas.devicePixelRatio);
            this.renderer.setSize(this.canvas.clientWidth, this.canvas.clientHeight);
            this.renderer.shadowMap.enabled = this.rendererParams.shadowMapEnabled
            this.renderer.shadowMap.type = this.rendererParams.shadowMapType
            this.renderer.toneMappingExposure = this.rendererParams.toneMappingExposure
            this.renderer.toneMapping = this.rendererParams.toneMapping
            this.renderer.gammaInput = this.rendererParams.gammaInput
            this.renderer.gammaFactor = this.rendererParams.gammaFactor
        },
        setScene() {
            // --- Name
            this.scene.name = this.sceneName;

            // --- Background
            // In order : posX, negX, posY, negY, posZ, negZ
            const background1 = this.cubeTextureLoader.load([
                './assets/backgrounds/background1-posX.png',
                './assets/backgrounds/background1-negX.png',
                './assets/backgrounds/background1-posY.png',
                './assets/backgrounds/background1-negY.png',
                './assets/backgrounds/background1-posZ.png',
                './assets/backgrounds/background1-negZ.png',
            ]);
            this.backgrounds.push(background1);
            this.scene.background = this.backgrounds[0];

            // --- Fog
            // We need to adapt the range of this input. The fog is not the same with a GLTF (Far model) and a STL (Close model)
            //TODO: These modif here ?
            if (this.format === 'STL') {
                this.fogDensity = 0.0045
                this.$refs.refInputFogDensity.min = 0.00002
                this.$refs.refInputFogDensity.max = 0.015
                this.$refs.refInputFogDensity.step = 0.00001
            } else if (this.format === 'glTF') {
                this.fogDensity = 0.00003
                this.$refs.refInputFogDensity.min = 0.000002
                this.$refs.refInputFogDensity.max = 0.0001
                this.$refs.refInputFogDensity.step = 0.000001
            }
        },
        setCamera() {
            this.camera.position.set(this.cameraPosition.x, this.cameraPosition.y, this.cameraPosition.z);
        },
        setLights() {
            // --- Hemispheric
            this.hLight.position.set(this.hLightPosition.x, this.hLightPosition.y, this.hLightPosition.z)
            this.scene.add(this.hLight);

            // --- Directional
            this.dLight.position.set(this.dLight.position.x, this.dLight.position.y, this.dLight.position.z);

            //TODO: Usefull ?
            // this.dLight.castShadow = true;
            // this.dLight.shadow.mapSize.width = 2048;
            // this.dLight.shadow.mapSize.height = 2048;
            //
            // this.dLight.shadow.camera.left = -50;
            // this.dLight.shadow.camera.right = 50;
            // this.dLight.shadow.camera.top = 50;
            // this.dLight.shadow.camera.bottom = -50;
            //
            // this.dLight.shadow.bias = - 0.0001;
            // this.dLight.position.multiplyScalar( 30 );

            this.scene.add(this.dLight);
            this.scene.add(this.dLight.target);
        },
        setControls() {
            this.controls.target.set(this.controlsTarget.x, this.controlsTarget.y, this.controlsTarget.z);
            this.controls.update();
        },
        setModelControls() {
            // compute the box that contains all the stuff
            // from root and below
            const box = new THREE.Box3().setFromObject(this.model);

            const boxSize = box.getSize(new THREE.Vector3()).length();
            const boxCenter = box.getCenter(new THREE.Vector3());

            // set the camera to frame the box
            this.updateCamera(boxSize * 0.5, boxSize, boxCenter, this.camera);

            // update the Trackball controls to handle the new size
            this.controls.maxDistance = boxSize * 10;
            this.controls.target.copy(boxCenter);
            this.controls.update();
        },
        setPostProcessing() {
            //TODO: update postprocessing ? re-addPass ?
            // // --- ShaderPass (?), SAO, SSAA
            // this.ssaaRenderPass.unbiased = false;
            // this.ssaaRenderPass.enabled = this.ssaaRenderPassEnabled
            // this.saoPass.enabled = this.saoPassEnabled
            // this.saoPass.params = this.saoPassParams
            // this.filmPass.enabled = this.filmPassEnabled
            // this.filmPass.params = this.filmPassParams
            // --- Adding to the composer ---
            this.composer.addPass(this.renderPass);
            this.composer.addPass(this.filmPass);
            this.composer.addPass(this.ssaaRenderPass);
            this.composer.addPass(this.saoPass);

            // this.shaderPass.params = this.shaderPassProps
            // this.ssaaPass.params = this.ssaaPassProps


            // this.composer.addPass(this.ssaaPass);
            // this.composer.addPass(this.shaderPass);
        },
        setStats() {
            this.stats.dom.style.position = 'absolute'
            this.stats.dom.style.right = 0
            this.stats.dom.style.left = 'unset'
            // TODO: don't use DOM ?
            document.querySelector('#viewer').appendChild(this.stats.dom);
        },
        // ============ UPDATERS =====================
        updateModel() {
            // --- Rotation
            if ((this.rotationEnbaled) && this.model) {
                // Just spinning
                this.model.rotation.y += this.rotationStep;
                this.wireframeLines.rotation.y += this.rotationStep;
            }
            // --- Model visibility
            if (this.model) this.model.visible = this.modelVisible
            // --- Anisotropy
            this.scene.traverse((child) => {           // Way to modify an element already added to the scene
                if (child instanceof THREE.Mesh && this.format === 'glTF') {
                    child.material.map.anisotropy = this.anisoLevel
                    this.model = child
                    //TODO: That
                }
            })
            // --- Wireframe
            if (this.wireframeEnabled) {
                this.scene.add(this.wireframeLines);
            } else {
                this.scene.remove(this.wireframeLines);
            }
        },
        // updateRender() {
        //     // this.canvas.clientWidth = this.canvas.clientWidth      // Depends of the css
        //     // this.canvas.clientHeight = this.canvas.clientHeight     // Depends of the css
        //     // this.renderer.setSize(this.canvas.clientWidth, this.canvas.clientHeight);
        //     this.render()
        // },
        updateScene() {
            // --- Background
            if (this.backgroundEnabled) {
                this.scene.background = this.backgrounds[0];
            } else {
                this.scene.background = new THREE.Color(this.bkgrndSceneColorHex);
            }
            // --- Fog
            this.fog.density = this.fogDensity
            if (this.fogEnabled) {
                this.scene.fog = this.fog
            } else {
                this.scene.fog = null
            }
        }
        ,
        updatePostProcessing() {
            // --- SSAA
            this.ssaaRenderPass.unbiased = false;
            this.ssaaRenderPass.enabled = this.ssaaRenderPassEnabled
            // --- SAO
            this.saoPass.enabled = this.saoPassEnabled
            this.saoPass.params = this.saoPassParams
            // --- Film
            this.filmPass.enabled = this.filmPassEnabled
            this.filmPass.params = this.filmPassParams
        },
        updateCamera(sizeToFitOnScreen, boxSize, boxCenter, camera) {
            const halfSizeToFitOnScreen = sizeToFitOnScreen * 0.5;
            const halfFovY = THREE.MathUtils.degToRad(camera.fov * .5);
            const distance = halfSizeToFitOnScreen / Math.tan(halfFovY);
            // compute a unit vector that points in the direction the camera is now
            // in the xz plane from the center of the box
            const direction = (new THREE.Vector3())
                    .subVectors(camera.position, boxCenter)
                    .multiply(new THREE.Vector3(1, 0, 1))
                    .normalize();
            // move the camera to a position distance units way from the center
            // in whatever direction the camera was from the center already
            camera.position.copy(direction.multiplyScalar(distance).add(boxCenter));
            // pick some near and far values for the frustum that
            // will contain the box.
            camera.near = boxSize / 100;
            camera.far = boxSize * 100;
            camera.updateProjectionMatrix();
            // point the camera to look at the center of the box
            camera.lookAt(boxCenter.x, boxCenter.y, boxCenter.z);
        }
        ,
        updateLight() {
            // --- Hemispheric
            this.dLight.visible = this.dLightVisible
            this.hLight.visible = this.hLightVisible
            // --- Directional
            this.dLight.position.set(this.dLightPosition.x, this.dLightPosition.y, this.dLightPosition.z);
            this.dLight.shadow.camera.far = this.cameraFar;
            // --- Helpers
            if (this.dLightVisible && this.dLightHelperVisible) this.scene.add(this.dLightHelper);
            else this.scene.remove(this.dLightHelper);
            if (this.hLightVisible && this.hLightHelperVisible) this.scene.add(this.hLightHelper);
            else this.scene.remove(this.hLightHelper);


        },
        updateStats() {
            // const statsLeft = this.canvas.clientWidth - this.stats.dom.style.width + 'px'
            if (this.isFullscreen) {
                this.stats.dom.style.position = 'fixed'
                this.stats.dom.style.right = 0
                this.stats.dom.style.left = 'unset'
            } else {
                this.stats.dom.style.position = 'absolute'
                this.stats.dom.style.right = 0
                this.stats.dom.style.left = 'unset'
            }
        }
        ,
        // ============ CLEANERS =====================
        cleanRenderer() {
            this.renderer.forceContextLoss()
            this.renderer.context = null
            this.renderer.domElement = null
            this.renderer = null
        }
        ,
        // ============ FRONT ========================
        toggleFullscreen() {
            this.isFullscreen = !this.isFullscreen
        }
        ,
        toggleGui() {
            this.isGuiShown = !this.isGuiShown
        }
        ,
        toggleSubmenu(e) {
            const siblings = e.target.parentElement.childNodes
            // eslint-disable-next-line no-console
            console.log(siblings)
            for (const sibling of siblings) {
                // eslint-disable-next-line no-console
                console.log(sibling)
                if (sibling.classList.contains('submenu')) {
                    sibling.classList.toggle('submenuOpened')
                }

                if (sibling.classList.contains('endSubmenu')) {
                    sibling.classList.toggle('shown')
                }
            }
        }
        ,
        // ============ TOOLS ========================
        isWebglSupported() {
            try {
                return !!(window.WebGLRenderingContext && (this.canvas.getContext('webgl') || this.canvas.getContext('experimental-webgl')));
            } catch (e) {
                return false;
            }
        }
        ,
        isResizeNeeded() {
            const needResize = this.canvas.width !== this.canvas.clientWidth || this.canvas.height !== this.canvas.clientHeight;
            if (needResize) {
                this.renderer.setSize(this.canvas.clientWidth, this.canvas.clientHeight, false);
            }
            return needResize;
        }
        ,
    }
}
</script>

<style scoped>
/*______________ Main ______________*/

#viewer {
    width: 90%;
    height: 850px;
    position: relative;
    font-family: BlinkMacSystemFont,-apple-system,"Segoe UI",Roboto,Oxygen,Ubuntu,Cantarell,"Fira Sans","Droid Sans","Helvetica Neue",Helvetica,Arial,sans-serif;
    font-size: 1em;
    font-weight: 400;
    margin: auto;

}

#rendererDomElement {
    z-index: 9999;
    min-height: 100%;
    max-height: 100%;
    min-width: 100%;
    max-width: 100%;
    width: 100%;
    height: 100%;
    display: block;
}

/*______________ GUI ______________*/
.gui {
    top: 26px;
    display: none;
    overflow-y: scroll;
    max-height: calc(100% - 46px);
    z-index: 10000;
    position: absolute;
    text-align: left;
    line-height: 1.5;
    padding: 10px;
    margin: 0;
    list-style: none;
    color: white;
    opacity: 0.5;
    background-color: #000000;
    width: 270px;
    height: auto;
    /*position: absolute;*/
}

/*______________ GUI submenus ______________*/
.submenu {
    display: none;
    visibility: hidden;
    height: 0;
    margin: 0;
    list-style: none;
    padding: 0;
    line-height: 1.8;
}

.endSubmenu {
    display: none;
    height: 4px;
    background-color: gray;
    width: 100%;
    margin: 4px 0 3px 0;
}

.submenuOpened {
    /*animation-duration: 10s;*/
    /*animation-name: openSubmenu;*/
    display: block;
    visibility: visible;
    height: 100%;
}

/*______________ GUI items ______________*/
.gui .guiItem {
    list-style: none;
    display: flex;
    flex-direction: column;
    margin: 0 0 5px 0;
}

.gui .labelInput {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.gui .labelInput .labelBold {
    font-weight: 700;
}

.gui .labelInput label,
.gui .labelInput input,
.gui .labelInput select {
    cursor: pointer;
}

.gui .labelInput select {
    border: none;
    width: 40%;
    background-color: white;
    font-weight: 500;
    font-style: italic;
}

.gui .labelInput option:disabled {
    color: red;
}

.gui .labelInput input {
    color: black;
    border: none;
    background-color: white;
}

/*______________ Buttons ______________*/
.btnFs {
    cursor: pointer;
    z-index: 10000;
    position: absolute;
    /*margin-right: 10%;*/
    left: calc(100% - 40px);
    bottom: 0;
    opacity: 0.5;
    background-color: #000000;
    text-align: center;
    vertical-align: middle;
    line-height: 40px;
    list-style: none;
    color: white;
    width: 40px;
    height: 40px;
    /*position: absolute;*/
}

.btnToggleSubmenu {
    font-weight: 700;
    height: 14px;
    line-height: 10px;
    background-color: gray;
    border: 0;
    margin: 6px 0 3px 0;
    cursor: pointer;
}

.btnToggleGui {
    cursor: pointer;
    color: white;
    position: absolute;
    opacity: 0.5;
    background-color: #000000;
    z-index: 10000;
    width: 290px;
    height: 25px;
}

/*______________ Fullscreen ______________*/
.canvasFullscreen {
    min-height: 100vh;
    max-height: 100vh;
    min-width: 100vw;
    max-width: 100vw;
    width: 100vw;
    height: 100vh;
    position: fixed;
    top: 0;
    left: 0;
}

.guiFullscreen {
    position: fixed;
    top: 26px;
    left: 0;
}

.btnFsFullscreen {
    position: fixed;
    bottom: 0;
    right: 0;
}

.btnTguiFullscreen {
    position: fixed;
    top: 0;
    left: 0;
}

/*______________ Flags ______________*/

.shown {
    display: block;
}

.guiShown {
    border-bottom: 1px solid white;
}

/*______________ Animations ______________*/
@keyframes openSubmenu {
    from {
        height: 0;
    }
    to {
        height: 100%;
    }
}

@keyframes closeSubmenu {
    from {
        display: block;
        visibility: visible;
        height: 100%;
    }
    to {
        display: none;
        visibility: hidden;
        height: 0;
    }
}

/*______________ Overrides to avoide weird behaviours ______________ (TODO: Fix it)*/

/*___ In chunk-vendors.98565a43 :
.content blockquote:not(:last-child), .content dl:not(:last-child), .content ol:not(:last-child), .content p:not(:last-child), .content pre:not(:last-child), .content table:not(:last-child), .content ul:not(:last-child) {
    margin-bottom: 1em;
}*/
.gui, .submenu {
    margin: 0 !important;
}

</style>