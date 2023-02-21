<template>
    <div class="page-panel" v-show="bShowEditor">
        <iframe 
            src="./drawio.html" 
            style="width:100%;height:100%" 
            ref="drawioframe"
            name="drawioframe"
        ></iframe>
    </div>
</template>

<script>

export default {
    name: 'Drawio',

    data() {
        return {
            bShowEditor: true
        }
    },

    created() {
        var oThis = this

        emitter.on('save-document', async () => {
            var oData = {
                command: "save",
            }

            window.frames.drawioframe.postMessage(JSON.stringify(oData));
        })

        emitter.on('load-document', async () => {
            var oResponse = await fetch('./blank.drawio')
            var sText = await oResponse.text()

            var oData = {
                command: "load",
                data: sText
            }

            window.frames.drawioframe.postMessage(JSON.stringify(oData));
        })
    }
}
</script>

<style>

</style>