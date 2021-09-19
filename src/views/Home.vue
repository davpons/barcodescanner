<template>
	<ion-page>
		<ion-header :translucent="true">
			<ion-toolbar>
				<ion-title>DLW Scanner</ion-title>
			</ion-toolbar>
		</ion-header>

		<ion-content :fullscreen="true" :style="{background: scanActive ? 'transparent': '#fff'}">
			<div id="container">
				<ion-card v-if="result && !scanActive">
					<ion-card-header>
						<ion-card-subtitle>Resultado de la exploración</ion-card-subtitle>
					</ion-card-header>
					<ion-card-content>{{ result }}</ion-card-content>
				</ion-card>
				<ion-button color="primary" @click="startScan()" v-if="!scanActive">
					Iniciar escaner
				</ion-button>
			</div>
			<div id="scanbuttons">
				<ion-button color="primary" @click="stopScan()" v-if="scanActive">
					Detener escaner
				</ion-button>
			</div>
			<div id="scanbox" v-if="scanActive">
				<div></div>
			</div>
		</ion-content>
	</ion-page>
</template>

<script lang="ts">
import { IonButton, IonContent, IonHeader, IonPage, IonTitle, IonToolbar, alertController } from '@ionic/vue';
import { defineComponent } from 'vue';
import { BarcodeScanner } from '@capacitor-community/barcode-scanner';

export default defineComponent({
	name: 'Home',
	components: {
		IonButton,
		IonContent,
		IonHeader,
		IonPage,
		IonTitle,
		IonToolbar,
	},
	data() {
		return {
			result: '',
			scanActive: false,
		}
	},
	methods: {
        async startScan() {
			const allowed = await this.checkPermission();
			if (allowed) {
				this.scanActive = true;
				await BarcodeScanner.hideBackground();
				const result = await BarcodeScanner.startScan();
				if (result.hasContent) {
					this.result = result.content ? result.content : '';
					this.scanActive = false;
				}
			}
        },
        async checkPermission() {
			// eslint-disable-next-line no-async-promise-executor
			return new Promise(async resolve => {
				const status = await BarcodeScanner.checkPermission({ force: true });
				if (status.granted) {
					resolve(true);
				} else if (status.denied) {
					this.presentAlertConfirm();
					resolve(false);
				} else {
					resolve(false);
				}
			})
        },
		async presentAlertConfirm() {
			const alert = await alertController
			.create({
				header: 'Permisos',
				message: 'Esta aplicación necesita acceso a la cámara del dispositivo',
				buttons: [
					{
						text: 'Cancelar',
						role: 'cancel',
						cssClass: 'secondary',
						handler: () => {
							console.log('Cancelado')
						},
					},
					{
						text: 'Aceptar',
						handler: () => {
							BarcodeScanner.openAppSettings();
						},
					},
				],
			});
			return alert.present();
		},
        async stopScan() {
			await BarcodeScanner.stopScan();
            await BarcodeScanner.showBackground();
			this.scanActive = false;
        },
        async prepare() {
            await BarcodeScanner.prepare();
        },
	},
    deactivated() {
        this.stopScan();
    },
    beforeUnmount() {
        this.stopScan();
    },
    mounted() {
        this.prepare();
    },
});
</script>

<style scoped>
#container {
	text-align: center;
	position: absolute;
	left: 0;
	right: 0;
	top: 50%;
	transform: translateY(-50%);
}
#container strong {
	font-size: 20px;
	line-height: 26px;
}
#container p {
	font-size: 16px;
	line-height: 22px;
	color: #8c8c8c;
	margin: 0;
}
#container a {
	text-decoration: none;
}
#scanbox {
	position: absolute;
	width: 305px;
	height: 305px;
	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);
	box-shadow: 0 0 0 100vmax rgba(0,0,0,.55);
}
#scanbox::before {
    content: '';
    position: absolute;
    top: -3px;
	left: -3px;
    border-top: 6px solid #fff;
	border-left: 6px solid #fff;
	border-top-left-radius: 5px;
    height: 50px;
    width: 50px;
}
#scanbox::after {
    content: '';
    position: absolute;
    top: -3px;
	right: -3px;
    border-top: 6px solid #fff;
	border-right: 6px solid #fff;
	border-top-right-radius: 5px;
    height: 50px;
    width: 50px;
}
#scanbox div::before {
    content: '';
    position: absolute;
    bottom: -3px;
	left: -3px;
    border-bottom: 6px solid #fff;
	border-left: 6px solid #fff;
	border-bottom-left-radius: 5px;
    height: 50px;
    width: 50px;
}
#scanbox div::after {
    content: '';
    position: absolute;
    bottom: -3px;
	right: -3px;
    border-bottom: 6px solid #fff;
	border-right: 6px solid #fff;
	border-bottom-right-radius: 5px;
    height: 50px;
    width: 50px;
}
#scanbuttons {
	position: absolute;
	bottom: 16px;
	left: 0;
	height: 40px;
	z-index: 1;
	text-align: center;
	width: 100%;
}
ion-content {
	--background: transparent;
}
</style>