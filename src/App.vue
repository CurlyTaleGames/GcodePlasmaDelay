<template>
  <div id="app" class="container py-5">
    <!-- Logo and header text -->
    <div class="text-center mb-5">
      <img src="@/assets/logo.png" alt="Logo" class="mb-3" style="max-width: 150px;">
      <h1 class="mb-3">Gcode Plasma Delay</h1>
      <p class="lead">
        Drag and drop your .gcode or .nc file into the area below to add a delay to the start and stop of cutting for your CNC plasma cutter.
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

    <!-- Drag and drop area -->
    <div class="drag-area border d-flex flex-column justify-content-center align-items-center mb-3" @click="triggerFileInput" @dragover.prevent @drop.prevent="handleDrop" style="height: 200px;">
      <i class="fas fa-file-upload fa-5x mb-4"></i>
      <p>Drag and drop your .gcode or .nc file here or <span class="file-input-label">click to browse</span></p>
      <input type="file" ref="fileInput" @change="handleFiles" style="display: none;" accept=".gcode, .nc">
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
        <h2>Getting Started with Plasma CNC</h2>
        <p>Follow these instructions to begin using your plasma CNC cutter:</p>
        <!-- Embedded YouTube video -->
        <div class="embed-responsive embed-responsive-16by9 mb-3">
          <iframe class="embed-responsive-item"
                  src="https://www.youtube.com/embed/MGpWirqkfZk"
                  allowfullscreen></iframe>
        </div>
        <!-- Links to software -->
        <ul>
          <li><a href="https://lasergrbl.com/" target="_blank">LaserGRBL Software</a> - for control and operation of your CNC.</li>
          <li><a href="https://inkscape.org/" target="_blank">Inkscape</a> - for creating and editing designs to be cut.</li>
        </ul>
        <p>First, create your design in Inkscape and save it as an SVG file. Then, import your SVG into LaserGRBL to convert your design into G-code tailored for your CNC machine. Make sure to configure the proper settings for your material and plasma cutter model. Always perform a test run to ensure everything is calibrated correctly.</p>
      </div>
    </div>


    <!-- Container for centering the button -->
    <div class="d-flex justify-content-center">
      <!-- Collapsible section button -->
      <button class="btn btn-primary" type="button" @click="toggleCollapse" :aria-expanded="!isCollapsed.toString()" aria-controls="cncInstructions">
        <span>{{ isCollapsed ? 'Learn How to Get Started with Plasma CNC' : 'Hide Instructions' }}</span>
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
    this.customDwellCommand = localStorage.getItem('customDwellCommand') || 'G04P2.0';
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
      return filename.endsWith('.gcode') || filename.endsWith('.nc');
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
      this.customDwellCommand = 'G04P2.0';
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
