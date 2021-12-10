# 1. recognize object
# 2. detect pose detail (by using vue)

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


# LIBs

    @tensorflow-models/mobilenet
    @tensorflow-models/posenet
    @tensorflow/tfjs
    bootstrap
    bootstrap-vue

# TASKS

1. CLASSIFY: use image classification to determine what is in a picture https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ0ucPLLnB4Pu1kMEs2uRZISegG5W7Icsb7tq27blyry0gnYhVOfg

    file: components/views/Home.vue (classifier models imported: ImageClassifier, and TensorInformation)

    await this.classifier.Classify(image);

    UI:
    

and 

2. POSE: use pose detection to draw the key points, such as the major joints and major facial landmarks of a person.

    file: components/views/Pose.vue
    imported: PoseClassifier, and Keypoint posenet tf models

    UI:




# UI

## CLASSIFY UI

file: App.vue provides UI layout, template:

    template>
    <div id="app">
        <b-navbar toggleable="lg" type="dark" variant="info">
        <b-collapse id="nav-collapse" is-nav>
            <b-navbar-nav>
            <b-nav-item to="/">Classifier</b-nav-item>
            <b-nav-item to="/pose">Pose</b-nav-item>
            </b-navbar-nav>
        </b-collapse>
        </b-navbar>
        <router-view/>
    </div>
    </template>

file: components/HelloWorld.vue

    <template>
    <div class="container">
        <img crossorigin="anonymous" id="img" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ0ucPLLnB4Pu1kMEs2uRZISegG5W7Icsb7tq27blyry0gnYhVOfg" alt="Dog" ref="dogId" >
        <div class="row">
        <div class="col">
            <b-list-group>
            <b-list-group-item v-for="tensor in tensors" v-bind:key="tensor.className">
                {{ tensor.className }} - {{ tensor.probability }}
            </b-list-group-item>
            </b-list-group>
        </div>
        </div>
    </div>
    </template>

    <script lang="ts">
    import {Component, Vue} from 'vue-property-decorator';
    import {ImageClassifier} from '@/Models/ImageClassifier';
    import {TensorInformation} from '@/Models/TensorInformation';

    @Component
    export default class HelloWorld extends Vue {
    private readonly classifier: ImageClassifier = new ImageClassifier();
    private tensors: TensorInformation[] | null = null;

    constructor() {
        super();
        this.Classify();
    }
    public Classify(): void {
        this.$nextTick().then(async () => {
        /* tslint:disable:no-string-literal */
        const dog = this.$refs['dogId'];
        /* tslint:enable:no-string-literal */
        if (dog !== null && !this.tensors) {
            const image = dog as HTMLImageElement;
            this.tensors = await this.classifier.Classify(image);
        }
        });
    }
    }
    </script>

views/Home.vue will import HelloWorld.vue UI component:

    <template>
    <div class="home">
        <HelloWorld />
    </div>
    </template>

    <script lang="ts">
    import { Component, Vue } from 'vue-property-decorator';
    import HelloWorld from '@/components/HelloWorld.vue'; // @ is an alias to /src

    @Component({
    components: {
        HelloWorld,
    },
    })
    export default class Home extends Vue {}
    </script>


## POSE UI

file: Pose.vue (contains component Pose embedded) and is referring the static image, for which different areas of body are detected.

    <template>
    <div class="container">
        <div class="outsideWrapper">
        <div class="insideWrapper">
            <img crossorigin="anonymous" class="coveredImage" id="img" src="https://www.yogajournal.com/.image/t_share/MTQ3MTUyNzM1MjQ1MzEzNDg2/mountainhp2_292_37362_cmyk.jpg" alt="Pose" ref="poseId" >
            <canvas ref="posecanvas" id="canvas" class="coveringCanvas" width="1200" height="675"></canvas>
        </div>
        </div>
        <div class="row">
        <div class="col">
            <b-table striped hover :items="keypoints" :fields="fields"></b-table>
        </div>
        </div>
    </div>
    </template>

    <script lang="ts">
    import { Component, Vue } from 'vue-property-decorator';
    import {PoseClassifier} from '@/Models/PoseClassifier';
    import {Keypoint} from '@tensorflow-models/posenet';
    @Component
    export default class Pose extends Vue {
        private readonly classifier: PoseClassifier = new PoseClassifier();
        private keypoints: Keypoint[] | null;
        private fields =
            {'score':
            { label: 'Confidence', sortable: true},
            'part':
            { label: 'Part', sortable: true},
            'position.x':
            {label:'X'},
            'position.y': {label: 'Y'}};

        constructor() {
        super();
        this.keypoints = null;
        this.Classify();
        }
        public Classify(): void {
        this.$nextTick().then(async () => {
            /* tslint:disable:no-string-literal */
            const pose = this.$refs['poseId'];
            const poseCanvas = this.$refs['posecanvas'];
            /* tslint:enable:no-string-literal */
            if (pose !== null) {
            const image: HTMLImageElement = pose as HTMLImageElement;
            const canvas: HTMLCanvasElement = poseCanvas as HTMLCanvasElement;
            this.keypoints = await this.classifier.Pose(image, canvas);
            }
        });
        }
    }
    </script>



# CLASSIFY APP CODE

Models/ImageClassifier.ts

    export class ImageClassifier {
        private model: MobileNet | null = null;

        constructor() {
            // If running on Windows, there can be issues loading WebGL textures properly.
            // Running the following command solves this.
            tf.ENV.set('WEBGL_PACK', false);
        }
        public async Classify(image: tf.Tensor3D | ImageData | HTMLImageElement | HTMLCanvasElement | HTMLVideoElement):
            Promise<TensorInformation[] | null> {
            if (!this.model) {
            this.model = await mobilenet.load();
            }
            
            // inference
            if (this.model) {
                const result = await this.model.classify(image);
                return {
                    ...result,
                };
            }
            return null;
        }   
    }

Models/TensorInformation.ts -> class and probability on inference - image recognition, transforms result from MobileNet inference into meaningful output info.

        export interface TensorInformation {
            className: string;
            probability: number;
        }


OUTPUT of image classification - recogonition:

    Border collie - 0.8018293380737305
    borzoi, Russian wolfhound - 0.10988148301839828
    collie - 0.05392720550298691

# POSE DETECTION APP CODE

file: Models/PoseClassifier.ts

    import * as tf from '@tensorflow/tfjs';
    import * as posenet from '@tensorflow-models/posenet';
    import {Keypoint, Pose, PoseNet} from '@tensorflow-models/posenet';
    import {DrawPose} from '@/Models/DrawPose';

    export class PoseClassifier {
        private model: PoseNet | null = null;
        private drawPose: DrawPose | null = null;
        constructor() {
            // If running on Windows, there can be issues loading WebGL textures properly.
            // Running the following command solves this.
            tf.ENV.set('WEBGL_PACK', false);
        }

        public async Pose(image: HTMLImageElement, canvas: HTMLCanvasElement): Promise<Keypoint[] | null> {
            if (!this.model) {
                this.model = await posenet.load();
                this.drawPose = new DrawPose(canvas);
            }

            if (this.model) {
                const result: Pose = await this.model.estimateSinglePose(image);
                if (result) {
                    this.drawPose!.Draw(result.keypoints);
                    return result.keypoints;
                }
            }
            return null;
        }
    }




Models/Draw.ts

        import {Keypoint} from '@tensorflow-models/posenet';

        export class DrawPose {
            constructor(private canvas: HTMLCanvasElement, private context = canvas.getContext('2d')) {
                this.context!.clearRect(0, 0, this.canvas.offsetWidth, this.canvas.offsetHeight);
                this.context!.fillStyle = '#ff0300';
            }

            public Draw(keys: Keypoint[]): void {
                keys.forEach((kp: Keypoint) => {
                this.context!.fillRect(kp.position.x - 2.5, kp.position.y - 2.5, 5, 5);
                });
            }
        }

# OUTPUT of POSE details detection on the static image

    Confidence	Part	X	Y
    0.000052599076298065484	nose	276.4745290926697	502.38496005004527
    0.000041501818486722186	leftEye	42.53337017400269	669.6606078558224
    0.00004082535087945871	rightEye	6.834875753315866	281.62938692449467
    0.00021380127873271704	leftEar	1059.3086744440345	454.6194942485685
    0.00004942010491504334	rightEar	1138.8086843249368	528.1598393924158
    0.001391554600559175	leftShoulder	1064.2790152692876	454.68975825550297
    0.0005532680079340935	rightShoulder	182.06104194937026	464.39221209517575
    0.0009933541296049953	leftElbow	19.30278244147212	604.2803008761534
    0.0005455551436170936	rightElbow	31.56488170913259	613.8550325388018
    0.0004648528411053121	leftWrist	1093.5073415103288	480.6876853241058
    0.000587177462875843	rightWrist	1082.918225653457	613.9265496228498
    0.003878373419865966	leftHip	1098.4242019524663	645.2928424235268
    0.0026300521567463875	rightHip	1105.9350462426223	643.3637149256133
    0.0006529011880047619	leftKnee	1096.895873405881	640.6914210107278
    0.0004500265058595687	rightKnee	72.84445802839771	568.6460274263377
    0.0006650629220530391	leftAnkle	1139.2335631192032	645.3642372561492
    0.0005961144343018532	rightAnkle	1073.346472188554	678.4396661141507


# PRODUCTION

        vue-ts-detect-pose>npm run build

        > detect-pose-vue-ts@1.0.0 build C:\SRC\React\TFJS\vue-ts-detect-pose
        > vue-cli-service build


        \  Building for production...Starting type checking and linting service...
        Using 1 worker with 2048MB memory limit
        -  Building for production...Browserslist: caniuse-lite is outdated. Please run next command `npm update`
        /  Building for production...

        WARNING  Compiled with 4 warnings                                                                        6:51:31 p.m.

        warning  in C:/SRC/React/TFJS/vue-ts-detect-pose/src/views/Pose.vue

        WARNING in C:/SRC/React/TFJS/vue-ts-detect-pose/src/views/Pose.vue
        31:16 missing whitespace
            29 |           { label: 'Part', sortable: true},
            30 |         'position.x':
        > 31 |           {label:'X'},
            |                ^
            32 |         'position.y': {label: 'Y'}};
            33 |
            34 |     constructor() {

        warning

        asset size limit: The following asset(s) exceed the recommended size limit (244 KiB).
        This can impact web performance.
        Assets:
        js/chunk-vendors.f74dc3af.js (1.6 MiB)

        warning

        entrypoint size limit: The following entrypoint(s) combined asset size exceeds the recommended limit (244 KiB). This can impact web performance.
        Entrypoints:
        app (1.8 MiB)
            css/chunk-vendors.48580aae.css
            js/chunk-vendors.f74dc3af.js
            css/app.f1a93efa.css
            js/app.f61c8cdc.js


        warning

        webpack performance recommendations:
        You can limit the size of your bundles by using import() or require.ensure to lazy load some parts of your application.
        For more info visit https://webpack.js.org/guides/code-splitting/

        File                                   Size              Gzipped

        dist\js\chunk-vendors.f74dc3af.js      1641.42 KiB       448.19 KiB
        dist\js\app.f61c8cdc.js                6.91 KiB          2.62 KiB
        dist\css\chunk-vendors.48580aae.css    191.67 KiB        26.73 KiB
        dist\css\app.f1a93efa.css              0.43 KiB          0.26 KiB

        Images and other types of assets omitted.

        DONE  Build complete. The dist directory is ready to be deployed.
        INFO  Check out deployment instructions at https://cli.vuejs.org/guide/deployment.html