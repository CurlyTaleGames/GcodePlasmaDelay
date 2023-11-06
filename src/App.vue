<template>
  <div id="app" class="container py-5">
    <!-- Logo and header text -->
    <div class="text-center mb-5">
      <img src="@/assets/logo.png" alt="Logo" class="mb-3" style="max-width: 150px;">
      <h1 class="mb-3">Gcode Plasma Delay</h1>
      <p class="lead">
        Add a delay to the start and stop of cutting for your plasma cutter CNC. CAM software such as LaserGRBL does not support this feature, so this tool will add the necessary commands to your G-code file.
      </p>
    </div>

     <!-- Dwell command input group with reset button -->
     <div class="input-group mb-3">
      <div class="input-group-prepend">
        <span class="input-group-text" id="dwellCommandAddon">Dwell Command</span>
      </div>
      <input type="text"
             class="form-control"
             placeholder="Enter dwell command"
             aria-label="DwellCommand"
             aria-describedby="dwellCommandAddon"
             v-model="customDwellCommand">
      <div class="input-group-append">
        <button class="btn btn-outline-secondary" type="button" @click="resetDwellCommand">Reset</button>
      </div>
    </div>

    <!-- Text explaining G04P2.0 command -->
    <p class="text-muted">
      Enter the dwell command in the format 'G04 P&lt;seconds&gt;', where &lt;seconds&gt; is the time the machine should pause. 
      For example, 'G04 P2.0' would make the machine pause for 2 seconds.
    </p>

    <!-- Drag and drop area -->
    <div class="drag-area border d-flex flex-column justify-content-center align-items-center mb-3" @click="triggerFileInput" @dragover.prevent @drop.prevent="handleDrop" style="height: 200px;">
      <i class="fas fa-file-upload fa-5x mb-4 mt-2"></i>
      <p>Drag and drop your .gcode, .nc, or .cnc file here or <span class="file-input-label">click to browse</span></p>
      <input type="file" ref="fileInput" @change="handleFiles" style="display: none;" accept=".gcode, .nc, .cnc">
    </div>

    <!-- Confirmation message -->
    <transition name="fade">
      <div class="alert alert-success text-center" v-if="showConfirmation" role="alert">
        File uploaded successfully! Check your downloads folder.
      </div>
    </transition>

     <!-- Collapsible section for plasma CNC instructions -->
     <div id="cncInstructions" class="collapse my-4">
      <div class="card card-body">
        <h2 class="mb-4">Converting a Laser CNC to a Plasma CNC</h2>
        <!-- Embedded YouTube video -->
        <div class="embed-responsive embed-responsive-16by9 mb-3">
          <iframe class="embed-responsive-item"
                  src="https://www.youtube.com/embed/MGpWirqkfZk"
                  allowfullscreen></iframe>
        </div>
        <p class="mb-4">There are plenty of cheap diode laser CNC kits available on Amazon. They are all pretty similar, but the one that I like the best is the <a href="https://www.amazon.com/dp/B09ZH6HGGZ">Longer RAY5 5W Laser Engraver</a>. It has a web interface to upload gcode files, and it can be run using a touch screen without a connection to a computer.</p>
        <!-- Links to software -->
        <h2>CNC Software</h2>
        <p>You'll need some software to create your design files and to cut out your parts.</p>
        <ul class="mb-4">
          <li><a href="https://inkscape.org/en/release/" target="_blank">Inkscape</a> - for creating and editing designs to be cut.</li>
          <li><a href="https://lasergrbl.com/download/" target="_blank">LaserGRBL Software</a> - for control and operation of your CNC.</li>
        </ul>
        <div class="instructions-container">
        <h2 class="instructions-title">Set the Cut Order</h2>
        <div class="video-wrapper mb-3">
          <video autoplay muted loop class="instruction-video w-100">
            <source src="@/assets/cut-order.mp4" type="video/mp4">
            Your browser does not support the video tag.
          </video>
        </div>
        <div class="instructions-text">
          <p class="mb-2">When cutting a part out of metal, you typically want to cut interior parts first and the exterior outline last.</p>
          <p class="mb-2">LaserGRBL will process your paths from the bottom to the top in Inkscape. To ensure the exterior outline is cut last, follow these steps:</p>
          <ol class="instructions-list mb-3">
            <li>Select all objects by navigating to <strong>Edit > Select All</strong>.</li>
            <li>Convert objects to paths by choosing <strong>Path > Objects to Path</strong>.</li>
            <li>Select the outline of your part, then raise it to the top using <strong>Object > Raise to Top</strong>.</li>
          </ol>
        </div>

        <div class="instructions-container">
        <h2 class="instructions-title">Convert SVG to Gcode</h2>
        <div class="video-wrapper mb-3">
          <video autoplay muted loop class="instruction-video w-100">
            <source src="@/assets/convert-to-nc.mp4" type="video/mp4">
            Your browser does not support the video tag.
          </video>
        </div>
        <div class="instructions-text">
          <p class="mb-2">Your CNC needs instructions to move the motors and turn the plasma cutter on and off.</p>
          <p class="mb-2">Use LaserGRBL to convert your Inkscape SVG to a Gcode/nc file by doing the following steps:</p>
          <ol class="instructions-list mb-3">
            <li>Load your SVG file by navigating to <strong>File > Open File</strong>.</li>
            <li>Leave the settings as default and click the <strong>Create!</strong> button.</li>
            <li>LaserGRBL will create the tool paths based on the SVG file. Save it by going to <strong>File > Save (Advanced)</strong>.</li>
            <li>Leave the settings as default and click the <strong>Save</strong> button.</li>
            <li>Use this web app to <strong>add delays to the file</strong></li>
          </ol>
        </div>
      </div>

</div>

      </div>
    </div>


    <!-- Container for centering the button -->
    <div class="d-flex justify-content-center">
      <!-- Collapsible section button -->
      <button class="btn btn-primary" type="button" @click="toggleCollapse" :aria-expanded="!isCollapsed.toString()" aria-controls="cncInstructions">
        <span>{{ isCollapsed ? 'Learn How to Get Started with a Plasma CNC' : 'Hide Instructions' }}</span>
        <i :class="['fas', isCollapsed ? 'fa-chevron-down' : 'fa-chevron-up', 'icon-spacing']"></i>
      </button>
    </div>



    
    <div class="container">
      <div class="row my-4">
        <div class="col text-center">
          <p>Finding this web app useful? Consider donating:</p>
          <a href="https://github.com/sponsors/CurlyTaleGames" target="_blank" class="btn btn-light mx-2">
            <img src="@/assets/github.svg" alt="Github Sponsors" height="22"/> GitHub Sponsors
          </a>
          <a href="https://www.paypal.com/donate/?hosted_button_id=L4GAK93DRELFW" target="_blank" class="btn btn-light mx-2">
            <img src="@/assets/PayPal.svg" alt="PayPal" height="22"/>
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { saveAs } from 'file-saver';

export default {
  name: 'App',
  data() {
    return {
      showConfirmation: false,
      customDwellCommand: '',
      isCollapsed: true,
    };
  },
  created() {
    // Get the custom dwell command from localStorage or use the default
    this.customDwellCommand = localStorage.getItem('customDwellCommand') || 'G04 P2.0';
  },
  methods: {
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    handleFiles(event) {
      const files = event.target.files;
      if (files.length > 0) {
        const file = files[0];
        if (this.isValidFile(file.name)) {
          this.processFile(file);
          this.showConfirmationMessage();
        } else {
          console.error("Invalid file extension");
        }
      }
    },
    handleDrop(event) {
      this.handleFiles({ target: { files: event.dataTransfer.files } });
    },
    isValidFile(filename) {
      return filename.endsWith('.gcode') || filename.endsWith('.nc') || filename.endsWith('.cnc');
    },
    processFile(file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      const content = e.target.result;
      const processedContent = this.addDwellTimes(content);
      const newFilename = this.generateNewFilename(file.name);
      this.downloadFile(processedContent, newFilename);

      // Reset the file input after processing
      this.resetFileInput();
    };
    reader.readAsText(file);
  },

  resetFileInput() {
    if (this.$refs.fileInput) {
      this.$refs.fileInput.value = '';
    }
  },
  addDwellTimes(content) {
    const lines = content.split('\n');
    const processedLines = lines.map(line => {
      if (line.trim().startsWith('S')) {
        return line + '\n' + this.customDwellCommand;
      }
      return line;
    });
    return processedLines.join('\n');
  },
    generateNewFilename(filename) {
  // Find the last dot to isolate the extension
  const lastDotIndex = filename.lastIndexOf('.');

  // If there's no dot, this means there's no extension; handle this as you see fit
  if (lastDotIndex === -1) return filename + '_plasma';

  // Extract the base name and the extension
  const base = filename.substring(0, lastDotIndex);
  const extension = filename.substring(lastDotIndex); // includes the dot

  // Return the new filename
  return `${base}_plasma${extension}`;
},
    downloadFile(content, filename) {
      const blob = new Blob([content], { type: 'text/plain;charset=utf-8' });
      saveAs(blob, filename);
    },
    showConfirmationMessage() {
      this.showConfirmation = true;
      setTimeout(() => {
        this.showConfirmation = false;
      }, 5000);
    },
    resetDwellCommand() {
      this.customDwellCommand = 'G04 P2.0';
      localStorage.setItem('customDwellCommand', this.customDwellCommand);
    },
    toggleCollapse() {
    this.isCollapsed = !this.isCollapsed;

    // Next we toggle the collapse state of the #cncInstructions element
    let cncInstructionsElement = document.getElementById('cncInstructions');
    let bsCollapse = new bootstrap.Collapse(cncInstructionsElement, {
      toggle: false
    });

    if (this.isCollapsed) {
      bsCollapse.hide();
    } else {
      bsCollapse.show();
    }
  },
  },
  // Update local storage whenever the custom dwell command changes
  watch: {
    customDwellCommand(newCommand) {
      localStorage.setItem('customDwellCommand', newCommand);
    },
  }
};
</script>

<style>
/* Drag area styles */
.drag-area {
  background-color: #5f236b;
  color: #fff;
  border: 2px dashed #5f236b;
  border-radius: 4px;
  padding: 20px;
  text-align: center;
  cursor: pointer;
  transition: border-color 0.3s;
}

.drag-area:hover {
  border-color: #813391;
  background-color: #813391;
}

/* Style for the clickable text to browse files */
.file-input-label {
  color: #b3a2c7;
  text-decoration: underline;
  cursor: pointer;
}

/* Fade transition for the confirmation message */
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0;
}

.alert {
  background-color: #be375f;
  border-color: #be375f;
  color: #fff;
}

.btn-outline-secondary{
  color: #be375f;
  border-color: #be375f;
}

.btn-outline-secondary:hover, .btn-outline-secondary:focus, .btn-outline-secondary:active, .btn-outline-secondary.active, .open > .dropdown-toggle.btn-outline-secondary{
  color: #fff;
  background-color: #be375f;
  border-color: #be375f;
}

.btn-primary {
  background-color: #be375f;
  border-color: #be375f;
}
.btn-primary:hover, .btn-primary:focus, .btn-primary:active, .btn-primary.active, .open > .dropdown-toggle.btn-primary {
  background-color: #da547c;
  border-color: #da547c;
}

#app .collapse {
  margin-top: 20px;
}

#app .card {
  border: 1px solid #ddd;
}

#app .card-body {
  padding: 20px;
}

#app .btn-primary {
  margin-top: 20px;
}

/* Responsive iframe for embedded video */
.embed-responsive {
  position: relative;
  display: block;
  width: 100%;
  padding: 0;
  overflow: hidden;
}

.embed-responsive::before {
  display: block;
  content: "";
}

.embed-responsive-16by9::before {
  padding-top: 56.25%;
}

.embed-responsive-item {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 0;
}

.icon-spacing {
  margin-left: 0.5rem; /* Adjust the space as needed */
}
</style>
