<template>
  <div>
  <button @click="launch">
    <slot></slot>
  </button>
  </div>
</template>

<script>
import { flatfileImporter } from "@flatfile/sdk";

export default {
  name: "flatfile-button",
  emits: [
    /**
     * Returns a `batchId` unique to the end user the token was generated for.
     * @see https://flatfile.com/docs/sdk/#init
     */
    "init",

    /**
     * Returns the same `batchId` generated during initialization (see `init`)
     * that is unique to the end user the token was generated for.
     * @see https://flatfile.com/docs/sdk/#launch-1
     */
    "launch",

    /**
     * Returns a JavaScript error object.
     * @see https://flatfile.com/docs/sdk/#error
     */
    "error",

    /**'
     * Returns a payload with a copy of the validated data from the upload.
     * @see https://flatfile.com/docs/sdk/#complete
     */
    "complete",
  ],
  props: {
    token: {
      type: String,
      required: true,
      validator: function (value) {
        return value && value.length;
      },
    },
    mountUrl: String,
    apiUrl: String,
    onUpload: Function,
    onClose: Function,
  },
  data: () => ({
    flatfileImporter: null,
    loaded: false,
    importerLoaded: true,
  }),
  mounted() {
    this.loadImporter();
  },
  methods: {
    loadImporter: function () {
      if (this.flatfileImporter) {
        return;
      }

      const tempImporter = flatfileImporter(this.token, {
        ...(this.mountUrl ? { mountUrl: this.mountUrl } : {}),
        ...(this.apiUrl ? { apiUrl: this.apiUrl } : {}),
      });

      tempImporter.on("init", this.onInit);
      tempImporter.on("launch", this.onLaunch);
      tempImporter.on("complete", this.onComplete);
      tempImporter.on("error", this.onError);

      if (typeof this.onUpload === "function") {
        tempImporter.on("upload", this.onUpload);
      }
      if (typeof this.onClose === "function") {
        tempImporter.on("close", this.onClose);
      }

      this.flatfileImporter = tempImporter;
      this.loaded = true;
    },
    launch: function () {
      this.validateInputs();

      this.flatfileImporter.launch().catch((e) => {
        if (typeof this.onError === "function") {
          this.onError(e);
        }
      });
    },
    validateInputs: function () {
      if (!this.token) {
        console.error("[Error] Flatfile VueJS Adapter - token not provided!");
        this.isImporterLoaded = false;
      }
    },
    onInit: function (batchId) {
      this.$emit("init", batchId);
    },
    onLaunch: function (batchId) {
      this.$emit("launch", batchId);
    },
    onError: function (error) {
      this.$emit("error", error);
    },
    onComplete: function (payload) {
      this.$emit("complete", payload);
    },
  },
};
</script>

<style></style>
