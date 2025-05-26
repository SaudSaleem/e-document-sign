<template>
  <div>
    <input type="file" accept="application/pdf" @change="onFileChange" />
    <canvas ref="pdfCanvas"></canvas>
  </div>
</template>

<script>
export default {
  name: 'PdfViewer',
  data() {
    return {
      pdfDoc: null,
      currentPage: 1,
    };
  },
  methods: {
    onFileChange(event) {
      const file = event.target.files[0];
      if (file && file.type === 'application/pdf') {
        const fileReader = new FileReader();
        fileReader.onload = (e) => {
          const typedArray = new Uint8Array(e.target.result);
          this.loadPdf(typedArray);
        };
        fileReader.readAsArrayBuffer(file);
      } else {
        alert('Please upload a valid PDF file.');
      }
    },
    async loadPdf(pdfData) {
      const loadingTask = window.pdfjsLib.getDocument({ data: pdfData });
      this.pdfDoc = await loadingTask.promise;
      this.renderPage(this.currentPage);
    },
    async renderPage(pageNum) {
      const page = await this.pdfDoc.getPage(pageNum);
      const viewport = page.getViewport({ scale: 1.5 });
      const canvas = this.$refs.pdfCanvas;
      const context = canvas.getContext('2d');

      canvas.height = viewport.height;
      canvas.width = viewport.width;

      const renderContext = {
        canvasContext: context,
        viewport: viewport,
      };
      await page.render(renderContext).promise;
    },
  },
};
</script>
