<script>
  import * as tf from "@tensorflow/tfjs";
  import * as KNNClassifierCreator from "@tensorflow-models/knn-classifier";
  import * as mobilenet from "@tensorflow-models/mobilenet";
  import Avatar from "./Avatar.svelte";
  import Loading from "./Loading.svelte";
  import { trainingComplete } from "./store.js";
  import { onMount } from "svelte";

  //***********************Variable declarations*****************************/
  let knnClassifier = KNNClassifierCreator.create();
  let switchState,
    webCamEl,
    webcam,
    mbNet,
    importing = true,
    adding = false,
    intervalStarted = false,
    currentAnimationFBX = null,
    resultantIndex = -1;
  const unsubscribe = trainingComplete.subscribe((el) => (switchState = el));
  var refreshInterval;
  let fileObj = {
    0: "Breakdance-Footwork-To-Idle.fbx",
    1: "Chapa-2.fbx",
    2: "Chicken-Dance.fbx",
    3: "Dancing.fbx",
    4: "Samba-Dancing.fbx",
    5: "Hip-Hop-Dancing.fbx",
  };
  let displayNames = [
    "Breakdance Footwork",
    "Chapa",
    "Chicken Dance",
    "Casual Boss",
    "Samba",
    "Hip Hop",
  ];

  //********************initialize training info************************/
  let trainingStatus = Object.keys(fileObj).map((key, index) => ({
    fileName: fileObj[key],
    displayName: displayNames[index],
    samplesAdded: 0,
  }));

  //**************************Mange training info state***************/
  let currentPosition = 0;
  const handleAdd = async () => {
    adding = true;
    await addTrainingData(displayNames[currentPosition]);
    trainingStatus[currentPosition] = {
      ...trainingStatus[currentPosition],
      samplesAdded: trainingStatus[currentPosition].samplesAdded + 1,
    };
    adding = false;
  };
  const handleNext = () => {
    currentPosition += 1;
  };
  const handleFinishTraining = () => {
    trainingComplete.update((el) => !el);
  };

  //***********************initialize everyting*************/
  const initialize = async () => {
    webCamEl = await document.getElementById("webcam");
    mbNet = await mobilenet.load();
    webcam = await tf.data.webcam(webCamEl, {
      resizeHeight: 1,
      resizeWidth: 1,
    });
    importing = false;
    //create a forever for loop for listening any kind of changes

    // setInterval(async () => {
    //   if (knnClassifier.getNumClasses() === displayNames.length) {
    //     const img = await webcam.capture();

    //     const activation = mbNet.infer(img, "conv_preds");

    //     const result = await knnClassifier.predictClass(activation);
    //     resultantIndex = result.classIndex;
    //     handleClick(result.classIndex.toString());
    //     img.dispose();
    //   }
    // }, 3000);
    await tf.nextFrame();
  };
  //predict
  async function startExecution() {
    if (knnClassifier.getNumClasses() === displayNames.length) {
      const img = await webcam.capture();

      const activation = mbNet.infer(img, "conv_preds");

      const result = await knnClassifier.predictClass(activation);
      resultantIndex = result.classIndex;
      handleClick(result.classIndex.toString());
      img.dispose();
    }
  }
  //start-stop interval

  const startInterval = () => {
    refreshInterval = setInterval(startExecution, 3000);
    intervalStarted = true;
  };
  const stopInterval = () => {
    clearInterval(refreshInterval);
    intervalStarted = false;
  };

  const handleRestart = () => {
    window.location.reload();
  };
  //add img -> tensor to the classifier
  const addTrainingData = async (className) => {
    const img = await webcam.capture();
    const activation = mbNet.infer(img, true);
    knnClassifier.addExample(activation, className);
    img.dispose();
  };

  //*************handle dropdownMenu click*************/
  const handleClick = (key) => {
    currentAnimationFBX = fileObj[key];
  };

  onMount(async () => initialize());
</script>

<div>
  <a href="https://github.com/ratul2f8" target="_blank"
   class="btn btn-dark btn-md"
   role="button"
   id={!switchState ? "github-other" : "github"}
  ><i class="fab fa-github"></i> View Profile</a>
  {#if importing}
    <Loading />
  {/if}
  <Avatar currentAnimationFBX={[currentAnimationFBX]} />
  <div class={switchState ? "training-holder-destroy" : "training-holder"}>
    <div class="bg-yellow">
      <h2>
        Train to recognize the step : <b>{displayNames[currentPosition]}</b>
      </h2>
      <h6>*Try to add as many samples as possible</h6>
      <br />
      <h3>Samples Added : {trainingStatus[currentPosition].samplesAdded}</h3>
    </div>
    <div class="button-holder">
      <button
        type="button"
        id="add-btn"
        class="btn btn-success btn-lg"
        disabled={adding}
        on:click={handleAdd}
      >
        {adding ? "Adding" : "Add Sample"}
      </button>
      {#if currentPosition !== trainingStatus.length - 1}
        <button
          type="button"
          class="btn btn-dark btn-lg"
          id="next-btn"
          on:click={handleNext}
          disabled={adding ||
            (!adding && trainingStatus[currentPosition].samplesAdded === 0)}
          >Next</button
        >
      {/if}
      {#if currentPosition === trainingStatus.length - 1}
        <button
          type="button"
          class="btn btn-dark btn-lg"
          id="next-btn"
          on:click={handleFinishTraining}
          disabled={adding ||
            (!adding && trainingStatus[currentPosition].samplesAdded === 0)}
          >Finish Training</button
        >
      {/if}
    </div>
  </div>
  <div class={switchState ? "camera-holder-main" : "camera-holder"}>
    <video
      playsinline
      autoplay
      muted
      class={switchState ? "webcam-action" : "webcam-train"}
      id="webcam"
      height="100%"
      width="100%"
    />
  </div>
  <div class="btn-group" id="topRight">
    <a
      class="btn btn-warning dropdown-toggle"
      href="#"
      role="button"
      id="dropdownMenuLink"
      data-bs-toggle="dropdown"
      aria-expanded="false"
    >
      Moves
    </a>
    <ul class="dropdown-menu" aria-labelledby="dropdownMenuLink">
      <li on:click={() => handleClick("0")}>
        <a class="dropdown-item" href="#">BreakDance FootWork</a>
      </li>
      <li on:click={() => handleClick("1")}>
        <a class="dropdown-item" href="#">Chapa</a>
      </li>
      <li on:click={() => handleClick("2")}>
        <a class="dropdown-item" href="#">Chicken</a>
      </li>
      <li on:click={() => handleClick("3")}>
        <a class="dropdown-item" href="#">Casual Boss</a>
      </li>
      <li on:click={() => handleClick("4")}>
        <a class="dropdown-item" href="#">Samba</a>
      </li>
      <li on:click={() => handleClick("5")}>
        <a class="dropdown-item" href="#">Hip Hop</a>
      </li>
    </ul>
  </div>
  <div
    class="btn-group"
    id="topRight-down"
    role="group"
    aria-label="Button group with nested dropdown"
  >
    <button
      type="button"
      class="btn btn-secondary"
      on:click={() => {
        if (intervalStarted) {
          stopInterval();
        } else {
          startInterval();
        }
      }}>{intervalStarted ? "Stop Execution" : "Start Execution"}</button
    >
    <button type="button" class="btn btn-danger" on:click={handleRestart}
      >Restart</button
    >
  </div>
  {#if resultantIndex !== -1 && intervalStarted}
    <div class="step-name">
      <h2>{displayNames[resultantIndex]}({resultantIndex + 1})</h2>
    </div>
  {/if}
</div>
