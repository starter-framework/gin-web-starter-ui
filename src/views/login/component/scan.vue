<template>
	<div class="login-scan-container">
		<div ref="qrcodeRef"></div>
	</div>
</template>

<script setup lang="ts">
import { onMounted, getCurrentInstance } from 'vue';
import QRCode from 'qrcodejs2-fixes';

const { proxy } = getCurrentInstance() as any;
// 初始化生成二维码
const initQrcode = () => {
	proxy.$refs.qrcodeRef.innerHTML = '';
	new QRCode(proxy.$refs.qrcodeRef, {
		text: `https://qm.qq.com/cgi-bin/qm/qr?k=RdUY97Vx0T0vZ_1OOu-X1yFNkWgDwbjC&jump_from=webapi`,
		width: 260,
		height: 260,
		colorDark: '#000000',
		colorLight: '#ffffff',
	});
};
// 页面加载时
onMounted(() => {
	initQrcode();
});
</script>

<style scoped lang="scss">
.login-scan-container {
	padding: 20px;
	display: flex;
	justify-content: center;
	animation: logoAnimation 0.3s ease;
}
</style>
