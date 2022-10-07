<script>
  import { fabric } from "fabric";
  import { onMount } from 'svelte';

  let canvas;
	let canv;
	const canvasWidth = 800
	const canvasHeight = 600
	let DrawingRectangle,rectangle, origX, origY, isDown = false;

	function zoomInHandler() {
		canv.setZoom(canv.getZoom() * 1.1);
		canv.renderAll();
	}

	function zoomOutHandler() {
		canv.setZoom(canv.getZoom() * 0.9);
		canv.renderAll();
	}

	function resetZoomHandler() {
		canv.setZoom(1);
		canv.renderAll();
	}

	function selectHandler() {
		DrawingRectangle = false;
		canv.selection = true;
		canv.off('mouse:down');
		canv.off('mouse:move');
		canv.off('mouse:up');
		changeSelectableStatus(true);
	}

	function deleteHanlder() {
		const active = canv.getActiveObject()
		if (active) {
			canv.remove(active)
			if (active.type == "activeSelection") {
				active.getObjects().forEach(targetRect => canv.remove(targetRect))
				canv.discardActiveObject().renderAll()
			}
  	}
	}
			
	function drawHandler() {
		canv.selection = false;
		draw();
		changeSelectableStatus(false);
	}

	function changeSelectableStatus(val) {
		canv.forEachObject(function(rect) {
			rect.selectable = val;
		})
		canv.renderAll();
	}

	function draw() {
		canv.on('mouse:down', onMouseDown);
		canv.on('mouse:move', onMouseMove);
		canv.on('mouse:up', onMouseUp);
	}

	function onMouseDown(o) {
		let pointer = canv.getPointer(o.e);
		console.log(pointer)
		isDown = true;
		origX = pointer.x;
		origY = pointer.y;


		rectangle = new fabric.Rect({
			left: origX,
			top: origY,
			fill: 'transparent',
			stroke: 'red',
			strokeWidth: 3,
			selectable: false
		});
		canv.add(rectangle);
	}

	function onMouseMove(o) {
		if (!isDown) return;
		let pointer = canv.getPointer(o.e);
		if (origX > pointer.x) {
			rectangle.set({
					left: Math.abs(pointer.x)
			});
		}
		if (origY > pointer.y) {
			rectangle.set({
					top: Math.abs(pointer.y)
			});
		}

		rectangle.set({
			width: Math.abs(origX - pointer.x)
		});
		rectangle.set({
			height: Math.abs(origY - pointer.y)
		});
		canv.renderAll();
	};

	function onMouseUp(o) {
		rectangle.setCoords();
		isDown = false;
		DrawingRectangle = false;
	};

	let imageUrl = "./public/screenChart.png";
	let background = new Image();
	background.src = imageUrl;	

  onMount(() => {
    canv = new fabric.Canvas(canvas,{preserveObjectStacking:true});

		const bgImg = new fabric.Image();
		bgImg.setSrc(imageUrl, function () {
			bgImg.set({
					top: 0,
					left: 0,
					scaleX: canvasWidth/bgImg.width,
					scaleY: canvasHeight/bgImg.height,
			});
		});
		canv.setBackgroundImage(bgImg);

    fabric.Image.fromURL("https://cdn.sstatic.net/Sites/stackoverflow/company/img/logos/so/so-icon.svg?v=6e4af45f4d66", function (oImg) {

			canv.sendToBack(oImg);
			oImg.on('mousedown', function() {
				oImg.centerPt = this.getCenterPoint();
				canv.forEachObject(function(obj) {
					obj.origPose = new fabric.Point(obj.left, obj.top);
					console.log(obj.origPose)
				})
			})

			oImg.on('mouseup', function() {
				delete this.centerPt;
				canv.forEachObject(function(obj) {
						delete obj.origPose;
				})
			})

			oImg.on('moving', function(evt) {
				let self = this;
				let diff = this.getCenterPoint().subtract(self.centerPt);
				canv.forEachObject(function(obj) {
					if (obj == self) return;
					obj.set({
						left: obj.origPose.x + diff.x,
						top: obj.origPose.y + diff.y
					})
					obj.setCoords();
				})
			})
			canv.renderAll();
		}, {
				selectable: false,
		});
  });

	
</script>

<style>
	.btnWrap {
		margin-top: 20px;
	}

	button {
		background-color: #ffc4c4;
		border: none;
		color: white;
		outline: none;
	}

</style>

<div class="container">
	<canvas bind:this={canvas} id="paper" width={canvasWidth} height={canvasHeight} style="border:1px solid #ccc;"></canvas>
	<div class="btnWrap">
		<button id="draw" on:click={drawHandler}>Draw ROI</button>
		<button id="select" on:click={selectHandler}>Select ROI(s)</button>
		<button id="zoomIn" on:click={zoomInHandler}>Zoom In</button>
		<button id="zoomOut" on:click={zoomOutHandler}>Zoom Out</button>
		<button id="btnResetZoom" on:click={resetZoomHandler}>Reset Zoom</button>
		<button id="delete" on:click={deleteHanlder}>Delete</button>
	</div>
	
</div> 

