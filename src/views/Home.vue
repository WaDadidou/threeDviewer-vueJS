<template>
    <div id="Home">

        <span class="uselessSpan">Add some cool html elements </span>

        <!--        <div class="dashboard">-->
        <!--            <div class="labelInput">-->
        <!--                <label for="fileUrl">Select your 3D file through an URL</label>-->
        <!--                &lt;!&ndash;                <div class="inputsContainer">&ndash;&gt;-->
        <!--                <input ref="refInputFileUrl" id="fileUrl" type="text"/>-->
        <!--&lt;!&ndash;                <div class="radiosContainer">&ndash;&gt;-->
        <!--&lt;!&ndash;                    <input id="radioGltf" type="radio"/>&ndash;&gt;-->
        <!--&lt;!&ndash;                    <label for="radioGltf">GLTF</label>&ndash;&gt;-->
        <!--&lt;!&ndash;                    <input id="radioSTL" type="radio"/>&ndash;&gt;-->
        <!--&lt;!&ndash;                    <label for="radioSTL">STL</label>&ndash;&gt;-->
        <!--&lt;!&ndash;                </div>&ndash;&gt;-->
        <!--                &lt;!&ndash;                    <input @change="getLocalFile" id="fileLocalPath" type="file"/>&ndash;&gt;-->
        <!--                &lt;!&ndash;                </div>&ndash;&gt;-->
        <!--            </div>-->
        <!--            <div class="btnsContainer">-->
        <!--                <div @click="displayModel" class="btn">Display</div>-->
        <!--                <div @click="cheat" class="btn">cheat</div>-->
        <!--            </div>-->
        <!--        </div>-->

        <!--        <Loader can-cancel="false" :active.sync="isLoading"/>-->
        <div class="uselessContainer">
            <span class="uselessSpan">Add some design, component or cat pictures</span>

            <ThreeDviewer
                    :key="threeDFilePath"
                    @on-load="onLoad"
                    v-if="threeDFilePath && (threeDformat === 'glTF' || threeDformat === 'STL')"
                    :threeDFilePath="threeDFilePath"
                    :format="threeDformat"
            />
            <span class="uselessSpan">Add buttons or stuff, whatever</span>
        </div>

    </div>
</template>

<script>
import ThreeDviewer from '../components/ThreeDviewer'
// import Loader from "../components/Loader";

export default {
    name: 'Home',
    components: {
        // Loader,
        ThreeDviewer
    },
    data() {
        return {
            isLoading: false,
            threeDFilePath: './assets/3Dmodels/Sainte_Victoire_Mountain.glb',        // Work only with this kind of model
            threeDformat: 'glTF',
        }
    },
    // created() {
    //     this.threeDformat = 'glTF'
    //     this.threeDFilePath = './assets/3Dmodels/Sainte_Victoire_Mountain.glb'
    // },
    methods: {
        deduceFormat() {
            if (this.threeDFilePath.includes('.stl')) {
                this.threeDformat = 'STL'
                return 'STL'
            } else if (this.threeDFilePath.includes('.gltf') || this.threeDFilePath.includes('.glb')) {
                this.threeDformat = 'glTF'
                return 'glTF'
            }
        },
        cheat() {
            this.isLoading = false;
        },
        onLoad() {
            this.isLoading = false;
        },
        displayModel() {
            this.isLoading = true;
            if (this.$refs.refInputFileUrl.value.includes('.stl')) this.threeDformat = 'STL'
            else if (this.$refs.refInputFileUrl.value.includes('.gltf') || this.$refs.refInputFileUrl.value.includes('.glb')) this.threeDformat = 'glTF'
            this.threeDFilePath = this.$refs.refInputFileUrl.value
        },
        getLocalFile(e) {
            this.threeDFilePath = e.target.value
        }

    }
}
</script>

<style scoped>
.dashboard {
    font-size: 1.1em;
    color: #222222;
    font-weight: bold;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.labelInput {
    display: flex;
    flex-direction: column;
    width: auto;
}

.labelInput label {
    margin-bottom: 10px;
}

.inputsContainer {
    display: flex;
    width: 100%;
    justify-content: space-between;
}

.radiosContainer {
    justify-content: space-evenly;
    margin-top: 20px;
    display: flex;
}


.inputsContainer #fileLocalPath {

}

.inputsContainer #fileUrl {
    width: 100%;
}

.btnsContainer {
    display: flex;
}

.btn {
    width: 100px;
    height: 50px;
    background-color: #049ef4;
    border-radius: 10px;
    cursor: pointer;
    margin: 20px;
    text-align: center;
    vertical-align: middle;
    line-height: 50px;
}

.btn:active {
    background-color: #0670ab;
}

.uselessContainer {
    display: flex;
}
.uselessSpan {
    color: #d78d00;
    font-weight: bold;
    font-size: 1.5em;
    line-height: 2.2;
}
.uselessContainer .uselessSpan{
    writing-mode: vertical-rl;
    text-orientation: mixed;
}
</style>
