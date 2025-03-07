<template>
	<div :style="{ width: w, height: h }" @touchmove.prevent>
		<canvas :id="uid" class="canvas" :data-uid="uid" :disabled="disabled"></canvas>
	</div>
</template>

<script>
import SignaturePad from 'signature_pad'
export default {
	name: "vueSignature",
	props: {
		sigOption: {
			type: Object,
			default: () => {
				return {
					backgroundColor: 'rgb(255,255,255)',
					penColor: 'rgb(0, 0, 0)'
				}
			},
		},
		w: {
			type: String,
			default: "100%"
		},
		h: {
			type: String,
			default: "100%"
		},
		clearOnResize: {
			type: Boolean,
			default: false
		},
		waterMark: {
			type: Object,
			default: () => {
				return {}
			}
		},
		disabled: {
			type: Boolean,
			default: false
		},
		defaultUrl: {
			type: String,
			default: ""
		}
	},
	data() {
		return {
			sig: () => { },
			option: {
				backgroundColor: 'rgb(255,255,255)',
				penColor: 'rgb(0, 0, 0)'
			},
			uid: ""
		}
	},
	watch: {
		disabled(val) {
			var _this = this
			if (_this.sig.off) {
				if (val) {
					_this.sig.off()
				} else {
					_this.sig.on()
				}
			}
		}
	},
	created() {
		var _this = this;
		this.uid = "canvas" + Math.random();
		var sigOptions = Object.keys(_this.sigOption);
		for (var item of sigOptions) {
			_this.option[item] = _this.sigOption[item]
		}
	},
	methods: {
		draw() {
			var _this = this;
			var canvas = document.getElementById(_this.uid)
			_this.sig = new SignaturePad(canvas, _this.option);
			_this.attachEventListeners();

			function resizeCanvas(c) {
				var url;
				if (!_this.isEmpty()) {
					url = _this.save();
				}
				var ratio = Math.max(window.devicePixelRatio || 1, 1);
				var reg = RegExp(/px/)
				c.width = reg.test(_this.w) ? _this.w.replace(/px/g, "") * ratio : c.offsetWidth * ratio;
				c.height = reg.test(_this.h) ? _this.h.replace(/px/g, "") * ratio : c.offsetHeight * ratio;
				c.getContext("2d").scale(ratio, ratio);
				_this.clear();
				!_this.clearOnResize && url !== undefined && _this.fromDataURL(url)
				Object.keys(_this.waterMark).length && _this.addWaterMark(_this.waterMark)
			}
			window.addEventListener("resize", resizeCanvas(canvas));
			resizeCanvas(canvas);

			if (_this.defaultUrl !== "") {
				_this.fromDataURL(_this.defaultUrl);
			}

			if (_this.disabled) {
				_this.sig.off();
			} else {
				_this.sig.on();
			}
		},
		clear() {
			var _this = this;
			_this.sig.clear();
		},
		save(format) {
			var _this = this;
			return format ? _this.sig.toDataURL(format) : _this.sig.toDataURL()
			// signaturePad.toDataURL(); // save image as PNG
			// signaturePad.toDataURL("image/jpeg"); // save image as JPEG
			// signaturePad.toDataURL("image/svg+xml"); // save image as SVG
		},
		fromDataURL(url) {
			var _this = this;
			_this.sig.fromDataURL(url)
		},
		isEmpty() {
			var _this = this;
			return _this.sig.isEmpty();
		},
		undo() {
			var _this = this;
			var data = _this.sig.toData();
			if (data) {
				data.pop()
				_this.sig.fromData(data);
			}
		},
		addWaterMark(data) {
			var _this = this;
			if (!(Object.prototype.toString.call(data) == '[object Object]')) {
				throw new Error("Expected Object, got " + typeof data + ".")
			} else {
				var vCanvas = document.getElementById(_this.uid);

				var textData = {
					text: data.text || '',
					x: data.x || 20,
					y: data.y || 20,
					sx: data.sx || 40,
					sy: data.sy || 40
				}

				var ctx = vCanvas.getContext('2d');
				ctx.font = data.font || '20px sans-serif';
				ctx.fillStyle = data.fillStyle || "#333";
				ctx.strokeStyle = data.strokeStyle || "#333";
				if (data.style == 'all') {
					ctx.fillText(textData.text, textData.x, textData.y);
					ctx.strokeText(textData.text, textData.sx, textData.sy);
				} else if (data.style == 'stroke') {
					ctx.strokeText(textData.text, textData.sx, textData.sy);
				} else {
					ctx.fillText(textData.text, textData.x, textData.y);
				}

				_this.sig._isEmpty = false
			}
		},
		attachEventListeners() {
			var _this = this;

			_this.sig.addEventListener('beginStroke', event => _this.$emit('beginStroke', event))
			_this.sig.addEventListener('endStroke', event => _this.$emit('endStroke', event))
			_this.sig.addEventListener('beforeUpdateStroke', event => _this.$emit('beforeUpdateStroke', event))
			_this.sig.addEventListener('afterUpdateStroke', event => _this.$emit('afterUpdateStroke', event))
		}
	},
	mounted() {
		var _this = this;
		this.$nextTick().then(() => {
			_this.draw()
		});
	}
}
</script>

<style>
canvas {
	width: 100%;
	height: 100%;
}
</style>
