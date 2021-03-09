import * as THREE from '//unpkg.com/three?module';

// Our input frames will come from here.
const videoElement =
    document.getElementsByClassName('input_video')[0];
const canvasElement =
    document.getElementsByClassName('output_canvas')[0];
const controlsElement =
    document.getElementsByClassName('control-panel')[0];
const canvasCtx = canvasElement.getContext('2d');

var signWord = "";
var lastLetter = "N/A";
var conTimes = 0;
var resetting = "";
var resetTimes = 0;

// Optimization: Turn off animated spinner after its hiding animation is done.
const spinner = document.querySelector('.loading');
spinner.ontransitionend = () => {
    spinner.style.display = 'none';
};

function onResults(results) {
    // Hide the spinner.
    document.body.classList.add('loaded');


    // Draw the overlays.
    canvasCtx.save();
    canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
    canvasCtx.drawImage(
        results.image, 0, 0, canvasElement.width, canvasElement.height);
    if (results.multiHandLandmarks && results.multiHandedness) {
        for (let index = 0; index < results.multiHandLandmarks.length; index++) {
            const classification = results.multiHandedness[index];
            const isRightHand = classification.label === 'Right';
            const landmarks = results.multiHandLandmarks[index];

            signLanguage(landmarks);

            drawConnectors(
                canvasCtx, landmarks, HAND_CONNECTIONS, {
                    color: isRightHand ? '#00FF00' : '#FF0000'
                });
            drawLandmarks(canvasCtx, landmarks, {
                color: isRightHand ? '#00FF00' : '#FF0000',
                fillColor: isRightHand ? '#FF0000' : '#00FF00'
            });
        }
    }
    canvasCtx.restore();
}

function signLanguage(landmarks) {
    var currLetter = isLetter(landmarks);
    var delay = 20;

    if (currLetter == "Resetting..."){
        resetTimes += 1;
        lastLetter = "_";
        resetting = "Resetting...";
    }
    else {
        resetTimes = 0;
        lastLetter = currLetter;
        resetting = "";
    }

    if (resetTimes > delay + 10) {
        signWord = "";
        resetTimes = 0;
    }

    if (currLetter == lastLetter && currLetter != "_")
        conTimes += 1;
    else
        conTimes = 0;

    if (conTimes > delay && currLetter != "Resetting...") {
        signWord += currLetter;
        conTimes = 0;
    }

    document.getElementById("sign_word").innerHTML = signWord;
    document.getElementById("sign_letter").innerHTML = lastLetter;
	  document.getElementById("resetting").innerHTML = resetting;
}

function isLetter(landmarks) {

    var thumb_str = isStraight(landmarks, "thumb");
    var index_str = isStraight(landmarks, "index");
    var middle_str = isStraight(landmarks, "middle");
    var ring_str = isStraight(landmarks, "ring");
    var pinky_str = isStraight(landmarks, "pinky");

    var thumb_cur = isCurled(landmarks, "thumb");
    var index_cur = isCurled(landmarks, "index");
    var middle_cur = isCurled(landmarks, "middle");
    var ring_cur = isCurled(landmarks, "ring");
    var pinky_cur = isCurled(landmarks, "pinky");

    var is_back = isBack(landmarks);
    var is_clamp = isClamp(landmarks);
    var is_pinch = isPinch(landmarks);

    /*Still missing:  
     * C
     * E
     * F
     * G
     * K
     * O
     * S
     * U
     * X
     */

    if (thumb_cur && index_cur && middle_cur && ring_cur && pinky_cur)
        return "Resetting..."
    else if (is_clamp)
        return "A"
    else if (thumb_str && index_cur && middle_cur && ring_cur && pinky_cur && is_back)
        return "B"
    else if (thumb_cur && index_str && middle_str && ring_str && pinky_str)
        return "D"
    else if (is_pinch && middle_str && ring_str && pinky_str && !is_back)
        return "H"
    else if (thumb_cur && index_cur && middle_cur && ring_cur && pinky_str)
        return "I"
    else if (thumb_str && index_str && middle_str && ring_str && pinky_str && !is_back)
        return "J"
    else if (thumb_cur && index_str && middle_cur && ring_cur && pinky_cur && !is_back)
        return "L"
    else if (thumb_cur && index_str && middle_str && ring_str && pinky_cur && !is_back)
        return "M"
    else if (thumb_cur && index_str && middle_str && ring_cur && pinky_cur && !is_back)
        return "N"
    else if (is_pinch && middle_str && ring_str && pinky_str && is_back)
        return "P"
    else if (thumb_str && index_cur && middle_cur && ring_cur && pinky_cur && !is_back)
        return "Q"
    else if (thumb_cur && index_cur && middle_str && ring_cur && pinky_cur)
        return "R"
    else if (thumb_str && index_str && middle_cur && ring_cur && pinky_cur)
        return "T"
    else if (thumb_cur && index_str && middle_str && ring_cur && pinky_cur && is_back)
        return "V"
    else if (thumb_cur && index_str && middle_str && ring_str && pinky_cur && is_back)
        return "W"
    else if (thumb_str && index_str && middle_str && ring_str && pinky_str && is_back)
        return "Y"
    else if (thumb_cur && index_str && middle_cur && ring_cur && pinky_cur && is_back) // Not working
        return "Z"
    else
        return "_"
}

function isPinch(landmarks) {
    var xDiff = Math.abs(landmarks[4].x - landmarks[8].x);
    var yDiff = Math.abs(landmarks[4].y - landmarks[8].y);
    var xLim = 0.06;
    var yLim = 0.06;

    return (xDiff < xLim && yDiff < yLim)
}

function isClamp(landmarks) {
    var xDiff = new Array(4);
    var yDiff = new Array(4);
    var xLim = 0.06;
    var yLim = 0.06;

    xDiff[0] = Math.abs(landmarks[4].x - landmarks[8].x);
    xDiff[1] = Math.abs(landmarks[4].x - landmarks[12].x);
    xDiff[2] = Math.abs(landmarks[4].x - landmarks[16].x);
    xDiff[3] = Math.abs(landmarks[4].x - landmarks[20].x);

    yDiff[0] = Math.abs(landmarks[4].y - landmarks[8].y);
    yDiff[1] = Math.abs(landmarks[4].y - landmarks[12].y);
    yDiff[2] = Math.abs(landmarks[4].y - landmarks[16].y);
    yDiff[3] = Math.abs(landmarks[4].y - landmarks[20].y);

    for (var i = 0; i < xDiff.length; i++)
        if (xDiff[i] > xLim)
            return false;

    for (var i = 0; i < yDiff.length; i++)
        if (yDiff[i] > yLim)
            return false;

    return true;
}

function isBack(landmarks) {
    var vectors = new Array(2);
    var axis = new THREE.Vector3(0, 0, 1);

    vectors[0] = new THREE.Vector3().subVectors(landmarks[0], landmarks[17]).normalize();
    vectors[1] = new THREE.Vector3().subVectors(landmarks[0], landmarks[5]).normalize();

    vectors[1] = vectors[1].applyAxisAngle(axis, Math.PI / 2);

    if (vectors[0].dot(vectors[1]) < 0)
        return true;
    else
        return false;
}

function isStraight(landmarks, finger) {
    var straight = false;
    var threshold = 0.75;
    var pt = new Array(4);
    var vectors = new Array(4);

    if (finger == "thumb") {
        pt[0] = 1;
        pt[1] = 2;
        pt[2] = 3;
        pt[3] = 4;
    } else if (finger == "index") {
        pt[0] = 5;
        pt[1] = 6;
        pt[2] = 7;
        pt[3] = 8;
    } else if (finger == "middle") {
        pt[0] = 9;
        pt[1] = 10;
        pt[2] = 11;
        pt[3] = 12;
    } else if (finger == "ring") {
        pt[0] = 13;
        pt[1] = 14;
        pt[2] = 15;
        pt[3] = 16;
    } else if (finger == "pinky") {
        pt[0] = 17;
        pt[1] = 18;
        pt[2] = 19;
        pt[3] = 20;
    }

    vectors[0] = new THREE.Vector3().subVectors(landmarks[0], landmarks[pt[0]]).normalize();
    vectors[1] = new THREE.Vector3().subVectors(landmarks[pt[0]], landmarks[pt[1]]).normalize();
    vectors[2] = new THREE.Vector3().subVectors(landmarks[pt[1]], landmarks[pt[2]]).normalize();
    vectors[3] = new THREE.Vector3().subVectors(landmarks[pt[2]], landmarks[pt[3]]).normalize();

    straight = (vectors[0].dot(vectors[1]) > threshold && vectors[1].dot(vectors[2]) > threshold && vectors[2].dot(vectors[3]) > threshold);

    return straight;
}

function isCurled(landmarks, finger) {
    var curled = false;
    var threshold = -0.3;
    var pt = new Array(4);
    var vectors = new Array(4);

    if (finger == "thumb") {
        pt[0] = 1;
        pt[1] = 2;
        pt[2] = 3;
        pt[3] = 4;
        threshold = 0.75;
    } else if (finger == "index") {
        pt[0] = 5;
        pt[1] = 6;
        pt[2] = 7;
        pt[3] = 8;
    } else if (finger == "middle") {
        pt[0] = 9;
        pt[1] = 10;
        pt[2] = 11;
        pt[3] = 12;
    } else if (finger == "ring") {
        pt[0] = 13;
        pt[1] = 14;
        pt[2] = 15;
        pt[3] = 16;
    } else if (finger == "pinky") {
        pt[0] = 17;
        pt[1] = 18;
        pt[2] = 19;
        pt[3] = 20;
    }

    vectors[0] = new THREE.Vector3().subVectors(landmarks[0], landmarks[pt[0]]).normalize();
    //vectors[1] = new THREE.Vector3().subVectors(landmarks[pt[0]], landmarks[pt[1]]).normalize();
    vectors[2] = new THREE.Vector3().subVectors(landmarks[pt[1]], landmarks[pt[2]]).normalize();
    vectors[3] = new THREE.Vector3().subVectors(landmarks[pt[2]], landmarks[pt[3]]).normalize();

    curled = (vectors[0].dot(vectors[2]) < threshold && vectors[0].dot(vectors[3]) < threshold);

    return curled;
}

const hands = new Hands({
    locateFile: (file) => {
        return `https://cdn.jsdelivr.net/npm/@mediapipe/hands@0.1/${file}`;
    }
});
hands.onResults(onResults);

/**
 * Instantiate a camera. We'll feed each frame we receive into the solution.
 */
const camera = new Camera(videoElement, {
    onFrame: async () => {
        await hands.send({
            image: videoElement
        });
    },
    width: 1280,
    height: 720
});
camera.start();

// Present a control panel through which the user can manipulate the solution
// options.
new ControlPanel(controlsElement, {
        selfieMode: true,
        maxNumHands: 1,
        minDetectionConfidence: 0.5,
        minTrackingConfidence: 0.5
    })
    .on(options => {
        videoElement.classList.toggle('selfie', options.selfieMode);
        hands.setOptions(options);
    });