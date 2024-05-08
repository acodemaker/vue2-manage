<template>
    <div>
        请扫码登陆
        <div id="qrcode"></div>
        <!--        <canvas ref="qrcodeCanvas"></canvas>-->
<!--        <el-input type="textarea" :value='qrCode' resize=true autosize = true></el-input>-->
    </div>
</template>

<script>
    import {getWxStatus} from '@/api/getData'
    import QRCode from 'qrcodejs2'  // 引入qrcode

    export default {
        data() {
            return {
                qrCode: null,

            }

        },
        created() {
            this.qrCode = this.$route.params.qrCode
            // require('qrcode-terminal').generate(qrcode, {small: true}
            // QRCode.toCanvas(this.$refs.qrcodeCanvas, this.qrCode, error => {
            //     if (error) console.error(error)
            //     console.log('QR Code generated successfully');
            // })
        },
        mounted() {
            this.qrcode();
          this.statusInterval =  setInterval(
                this.getWxStatus, 2000);
        },

        methods: {
            qrcode(){
                let qrcode = new QRCode('qrcode',{
                    width: 132,
                    height: 132,
                    text: this.qrCode,
                    colorDark: "#000",
                    colorLight: "#fff"
                })
            },
            async  getWxStatus() {
                const res = await getWxStatus()
                console.log('res Status : ' + JSON.stringify(res))
                if (res.code == 200) {
                    this.$router.push('message')
                    clearInterval(this.statusInterval)
                }
                // return res;
            }
        }

    }
</script>

<style scoped>

</style>
