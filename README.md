# js-
js通用

预加载模块

```js
!function (undefined) {
    var PreLoad = function (source, opt) {
        var cfg = opt || {};
        this.source = source;
        this.count = 0;
        this.total = source.length;
        // 加载事件
        this.onload = cfg.onload;
        this.prefix = cfg.prefix || "";
        this.init();
    };
    PreLoad.prototype.init = function () {
        var self = this;
        self.source.forEach(function (item) {
            var type = item.substr(item.lastIndexOf(".") + 1).toLowerCase(),
                src = self.prefix + item;
            switch (type) {
                case"js":
                    self.script.call(self, src);
                    break;
                case"css":
                    self.stylesheet.call(self, src);
                    break;
                case"svg":
                case"jpg":
                case"gif":
                case"png":
                case"jpeg":
                    self.image.call(self, src);
            }
        })
    };
    PreLoad.prototype.getProgress = function () {
        return Math.round(this.count / this.total * 100)
    };
    PreLoad.prototype.image = function (src) {
        var el = document.createElement("img");
        this.load(el, src);
        el.src = src;
    };
    PreLoad.prototype.stylesheet = function (src) {
        var el = document.createElement("link");
        this.load(el, src);
        el.rel = "stylesheet";
        el.type = "text/css";
        el.href = src;
        document.head.appendChild(el);
    };
    PreLoad.prototype.script = function (src) {
        var el = document.createElement("script");
        this.load(el, src);
        el.type = "text/javascript";
        el.src = src;
        document.head.appendChild(el);
    };
    PreLoad.prototype.load = function (el, src) {
        var self = this;
        el.onload = el.onerror = el.onabort = function (event) {
            self.onload && self.onload({count: ++self.count, total: self.total, item: src, type: event.type});
        }
    };

    // todo tasks
    var tasks = ['all img name'];
    var $progress = document.getElementById('j_progress');

    function loading(load) {
        var count = load.count;
        var total = load.total;
        $progress && ($progress.innerHTML = Math.round(100 * count / total) + '%');
        if (count === total) return complete();
    }

    function next(el, fn) {
        el.className += ' scaleOut';
        setTimeout(function () {
            el.parentNode.removeChild(el);
            fn && fn();
        }, 800)
    }

    function complete() {
        var $loader = document.getElementById('j_loader');
        var url = '';
        //renderHtml();
        next($loader, function () {
            if (!url) return;
            var script = document.createElement('script');
            script.src = url;
            document.body.appendChild(script);
        });
    }
     new PreLoad(tasks, {onload: loading, prefix: 'images/'});
    }();
```

###carmer.js
```js
var canvas = document.getElementById("myCanvas");
var ctx = canvas.getContext("2d");
var canTouch = true;
var canvasStyleWidth = parseInt(canvas.width);
var scaleFactor = canvas.width / canvasStyleWidth;
var viewableArea = canvasStyleWidth * scaleFactor;
var currentSelection;
var relativeMouse;
var relativeTouch1;
var relativeTouch2;
var pointerOn = false;
var mouseDown = false;
var mouseMoving = false;
var touches = [];
var fingerSize = 24;
var touch = {
	touch1PosX: 0,
	touch1PosY: 0,
	touch2PosX: 0,
	touch2PosY: 0,
	initAngle: 0,
	angle: 0,
	angleChange: 0,
	initLength: 0,
	length: 0,
	fingerLength: 0,
	lengthChange: 0
};
var image1 = new Image();
var image2 = new Image();
image2.src = '/Tpl/Wap/starbucks/imgs/camera_logo.png';
image2.onload = function() {
	image2.xPos = canvas.width - image2.width + 60;
	image2.yPos = canvas.height - image2.height;
	image2.initWidth = image2.width;
	image2.initHeight = image2.height;
	image2.currentWidth = image2.width;
	image2.currentHeight = image2.height;
	image2.initAngle = 0;
	image2.angle = 0;
	//update(image2);
};



function fileChange(e) {
	var f = e.files[0]; //涓?鍙笂浼?涓枃浠讹紝鍏跺疄鍙互涓婁紶澶氫釜鐨?
	var FR = new FileReader();
	FR.onload = function(f) {
		compressImg(this.result, 640);
	};
	FR.readAsDataURL(f); //鍏堟敞鍐宱nload锛屽啀璇诲彇鏂囦欢鍐呭锛屽惁鍒欒鍙栧唴瀹规槸绌虹殑
}

function compressImg(imgData, maxWidth) {
	if (!imgData) return;
	image1.onload = function() {
		image1.xPos = canvas.width / 2;
		image1.yPos = canvas.height / 2;
		image1.initWidth = image1.width;
		image1.initHeight = image1.height;
		image1.currentWidth = scale(image1.initHeight, viewableArea, image1.initWidth);
		image1.currentHeight = viewableArea;
		//alert(image1.initWidth+','+image1.currentHeight);
		if (image1.currentWidth < viewableArea) {
			image1.currentWidth = viewableArea;
			image1.currentHeight = scale(image1.initWidth, viewableArea, image1.initHeight);
		}
		//alert(image1.currentWidth+','+image1.currentHeight);
		image1.initAngle = 0;
		image1.angle = 0;
		update(image1);
	};
	image1.src = imgData;
}

function getCurrentSelection(position) {
	if (position.clientX > image2.xPos && position.clientX < image2.xPos + image2.initWidth && position.clientY > image2.yPos && position.clientY - 185 < image2.yPos + image2.initHeight) {
		return image2;
	}
	return image1;
}

canvas.addEventListener("touchstart", function(event) {
	pointerStart(event);
});
canvas.addEventListener("touchmove", function(event) {
	pointerMove(event);
});
canvas.addEventListener("touchend", function(event) {
	pointerEnd(event);
});
canvas.addEventListener("touchcancel", function(event) {
	pointerEnd(event);
});

function scale(oldSize, newSize, other) {
	var scaleFactor = newSize / oldSize;
	return other * scaleFactor;
}

function pointerStart(event) {
	if (!canTouch) return;
	currentSelection = getCurrentSelection(event.touches[0]);
	var relativeTouch1;
	var relativeTouch2;

	// single touch
	if (event.touches !== undefined && event.touches.length === 1) {
		relativeTouch1 = getRelative(event.touches[0]);
		//position
		touch.touch1PosX = relativeTouch1.x;
		touch.touch1PosY = relativeTouch1.y;
		touch.offsetX = touch.touch1PosX - currentSelection.xPos;
		touch.offsetY = touch.touch1PosY - currentSelection.yPos;
	}
	// multi touch
	else if (event.touches !== undefined && event.touches.length > 1) {
		currentSelection.initAngle = currentSelection.angle;
		currentSelection.initWidth = currentSelection.currentWidth;
		currentSelection.initHeight = currentSelection.currentHeight;
		relativeTouch1 = getRelative(event.touches[0]);
		relativeTouch2 = getRelative(event.touches[1]);

		//position
		touch.touch1PosX = relativeTouch1.x;
		touch.touch1PosY = relativeTouch1.y;
		touch.touch2PosX = relativeTouch2.x;
		touch.touch2PosY = relativeTouch2.y;

		var mid = findMidPoint(relativeTouch1, relativeTouch2);
		touch.offsetX = mid.x - currentSelection.xPos;
		touch.offsetY = mid.y - currentSelection.yPos;

		//angle
		touch.initAngle = slopeAngle(relativeTouch1, relativeTouch2);
		touch.angle = slopeAngle(relativeTouch1, relativeTouch2);
		touch.angleChange = 0;
		//length
		touch.initLength = findLength(relativeTouch1, relativeTouch2);
		touch.lengthChange = 0;
	}
}

function pointerMove(event) {
	if (!canTouch) return;

	var relativeTouch1;
	var relativeTouch2;
	event.preventDefault();
	touches = event.touches;
	// single touch
	if (event.touches !== undefined && event.touches.length === 1) {
		relativeTouch1 = getRelative(event.touches[0]);
		//position
		touch.touch1PosX = relativeTouch1.x;
		touch.touch1PosY = relativeTouch1.y;
		relativeTouch1 = {
			x: touch.touch1PosX - touch.offsetX,
			y: touch.touch1PosY - touch.offsetY
		};
		moveImage(currentSelection, relativeTouch1);
	}
	// multi touch
	if (event.touches !== undefined && event.touches.length > 1) {
		relativeTouch1 = getRelative(event.touches[0]);
		relativeTouch2 = getRelative(event.touches[1]);

		//position
		touch.touch1PosX = relativeTouch1.x;
		touch.touch1PosY = relativeTouch1.y;
		touch.touch2PosX = relativeTouch2.x;
		touch.touch2PosY = relativeTouch2.y;
		//angle
		touch.angle = slopeAngle(relativeTouch1, relativeTouch2);
		touch.angleChange = touch.angle - touch.initAngle;

		//length
		touch.length = findLength(relativeTouch1, relativeTouch2);
		touch.lengthChange = touch.length - touch.initLength;

		relativeTouch1 = {
			x: touch.touch1PosX,
			y: touch.touch1PosY
		};
		relativeTouch2 = {
			x: touch.touch2PosX,
			y: touch.touch2PosY
		};
		var mid = findMidPoint(relativeTouch1, relativeTouch2);

		twoFingerRotate(currentSelection, mid);
		twoFingerResize(currentSelection, mid);
		mid.x -= touch.offsetX;
		mid.y -= touch.offsetY;
		moveImage(currentSelection, mid);
	}

	// Mouse
	if (mouseDown) {
		relativeMouse = {
			x: touch.touch1PosX - touch.offsetX,
			y: touch.touch1PosY - touch.offsetY
		};
		moveImage(currentSelection, relativeMouse);
	}
	update(currentSelection);
}

function pointerEnd(event) {
	if (!canTouch) return;

	currentSelection.initAngle = currentSelection.angle;
	currentSelection.initWidth = currentSelection.currentWidth;
	currentSelection.initHeight = currentSelection.currentHeight;

	//position
	//angle
	touch.angle = 0;
	touch.angleChange = 0;
	//length
	touch.length = 0;
	touch.lengthChange = 0;

	if (event !== undefined) {
		if (event.touches !== undefined && event.touches.length == 1) {
			relativeTouch1 = getRelative(event.touches[0]);
			touch.touch1PosX = relativeTouch1.x;
			touch.touch1PosY = relativeTouch1.y;
			touch.offsetX = touch.touch1PosX - currentSelection.xPos;
			touch.offsetY = touch.touch1PosY - currentSelection.yPos;
		}
	}
}

function moveImage(image, location) {
	if (isInsideImage(image, location)) {
		image.xPos = location.x;
		image.yPos = location.y;
	}
}

function getRelative(position) {
	return {
		x: makeRelative(position).x,
		y: makeRelative(position).y
	};
}

function makeRelative(object) {
	var relativeCoords;
	//touch
	if (typeof object.clientX !== "undefined") {
		relativeCoords = {
			x: (object.clientX - canvas.getBoundingClientRect().left) * scaleFactor,
			y: (object.clientY - canvas.getBoundingClientRect().top) * scaleFactor
		};
		// mouse
	} else {
		relativeCoords = {
			x: (object.x - canvas.getBoundingClientRect().left) * scaleFactor,
			y: (object.y - canvas.getBoundingClientRect().top) * scaleFactor
		};
	}
	return relativeCoords;
}

function isInsideImage(image, pointer) {
	return isInside(image.xPos - image.currentWidth / 2, image.yPos - image.currentHeight / 2, image.currentWidth, image.currentHeight, pointer.x, pointer.y);
}

function isInside(x1, y1, width1, height1, x2, y2) {
	return x2 >= x1 && x2 < x1 + width1 && y2 >= y1 && y2 < y1 + height1;
}


function drawRotatedImage(image) {
	// $('#debug').html('');
	// $('#debug').append('main x:'+parseInt(image1.xPos - (image1.currentWidth / 2))+',y:'+parseInt(image1.yPos - (image1.currentHeight / 2))+',w:'+image1.currentWidth+',h:'+image1.currentHeight);
	// $('#debug').append('<br>');
	// $('#debug').append('logo x:'+parseInt(image2.xPos - (image2.currentWidth / 2))+',y:'+parseInt(image2.yPos - (image2.currentHeight / 2))+',w:'+image2.currentWidth+',h:'+image2.currentHeight);
	if (image == image2) {
		ctx.save();
		ctx.translate(image1.xPos, image1.yPos);
		ctx.rotate(image1.angle);
		ctx.translate(-image1.xPos, -image1.yPos);
		drawImageIOSFix(ctx, image1, 0, 0, image1.width, image1.height, (image1.xPos - (image1.currentWidth / 2)), (image1.yPos - (image1.currentHeight / 2)), image1.currentWidth, image1.currentHeight);
		ctx.restore();
	}

	ctx.save();
	ctx.translate(image.xPos, image.yPos);
	ctx.rotate(image.angle);
	ctx.translate(-image.xPos, -image.yPos);
	if (image == image1) {
		drawImageIOSFix(ctx, image, 0, 0, image.width, image.height, (image.xPos - (image.currentWidth / 2)), (image.yPos - (image.currentHeight / 2)), image.currentWidth, image.currentHeight);
	} else {
		ctx.drawImage(image, 0, 0, image.width, image.height, (image.xPos - (image.currentWidth / 2)), (image.yPos - (image.currentHeight / 2)), image.currentWidth, image.currentHeight);
	}

	ctx.restore();


	if (image == image1) {
		ctx.save();
		ctx.translate(image2.xPos, image2.yPos);
		ctx.rotate(image2.angle);
		ctx.translate(-image2.xPos, -image2.yPos);
		ctx.drawImage(image2, 0, 0, image2.width, image2.height, image2.xPos - (image2.currentWidth / 2), image2.yPos - (image2.currentHeight / 2), image2.currentWidth, image2.currentHeight);
		//drawImageIOSFix(ctx,image2, 0, 0, image2.width, image2.height, image2.xPos - (image2.currentWidth / 2), image2.yPos - (image2.currentHeight / 2), image2.currentWidth, image2.currentHeight);
		ctx.restore();
	}
}


function findLength(start, end) {
	var a = end.x - start.x;
	var b = end.y - start.y;
	var csq = (a * a) + (b * b);
	return Math.floor(Math.sqrt(csq));
}

function findMidPoint(start, end) {
	return {
		x: (start.x + end.x) / 2,
		y: (start.y + end.y) / 2
	};
}

function slopeAngle(start, end) {
	var run = end.x - start.x;
	var rise = end.y - start.y;
	return Math.atan2(run, rise);
}


function resizeImage(image, newWidth) {
	var origHeight = image.currentHeight;
	var origWidth = image.currentWidth;

	if (newWidth < 100) {
		newWidth = 100;
	} else {
		newWidth = newWidth;
	}
	image.currentWidth = newWidth;
	image.currentHeight = (origHeight / origWidth) * newWidth;
}

function twoFingerResize(image, location) {
	if (isInsideImage(image, location)) {
		touch.lengthChange = touch.length - touch.initLength;
		resizeImage(image, image.initWidth + touch.lengthChange);
	}
}

function twoFingerRotate(image, location) {
	if (isInsideImage(image, location)) {
		image.angle = image.initAngle - touch.angleChange;
	}
}

function update(image) {
	ctx.clearRect(0, 0, canvas.width, canvas.height);
	drawRotatedImage(image);
}

/*婊ら暅璋冩暣*/

var brighterMatrix = [
1, 0, 0, 0, 30,
0, 1, 0, 0, 30,
0, 0, 1, 0, 30,
0, 0, 0, 1, 0, ];

var darkerMatrix = [
1, 0, 0, 0, -30,
0, 1, 0, 0, -30,
0, 0, 1, 0, -30,
0, 0, 0, 1, 0, ];

var identityMatrix = [
1, 0, 0, 0, 0,
0, 1, 0, 0, 0,
0, 0, 1, 0, 0,
0, 0, 0, 1, 0, ];

function loadColorMatrix(matrix) {
	var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
	ctx.putImageData(colorMatrixFilter(imageData, matrix), 0, 0);
}

function colorMatrixFilter(pixels, m) {
	var d = pixels.data;
	for (var i = 0; i < d.length; i += 4) {
		var r = d[i];
		var g = d[i + 1];
		var b = d[i + 2];
		var a = d[i + 3];

		d[i] = r * m[0] + g * m[1] + b * m[2] + a * m[3] + m[4];
		d[i + 1] = r * m[5] + g * m[6] + b * m[7] + a * m[8] + m[9];
		d[i + 2] = r * m[10] + g * m[11] + b * m[12] + a * m[13] + m[14];
		d[i + 3] = r * m[15] + g * m[16] + b * m[17] + a * m[18] + m[19];
	}
	return pixels;
};



/**
 * Detecting vertical squash in loaded image.
 * Fixes a bug which squash image vertically while drawing into canvas for some images.
 * This is a bug in iOS6 devices. This function from https://github.com/stomita/ios-imagefile-megapixel
 * 
 */
function detectVerticalSquash(img) {
	var iw = img.naturalWidth,
		ih = img.naturalHeight;
	var canvas = document.createElement('canvas');
	canvas.width = 1;
	canvas.height = ih;
	var ctx = canvas.getContext('2d');
	ctx.drawImage(img, 0, 0);
	var data = ctx.getImageData(0, 0, 1, ih).data;
	// search image edge pixel position in case it is squashed vertically.
	var sy = 0;
	var ey = ih;
	var py = ih;
	while (py > sy) {
		var alpha = data[(py - 1) * 4 + 3];
		if (alpha === 0) {
			ey = py;
		} else {
			sy = py;
		}
		py = (ey + sy) >> 1;
	}
	var ratio = (py / ih);
	return (ratio === 0) ? 1 : ratio;
}

/**
 * A replacement for context.drawImage
 * (args are for source and destination).
 */
function drawImageIOSFix(ctx, img, sx, sy, sw, sh, dx, dy, dw, dh) {
	var vertSquashRatio = detectVerticalSquash(img);
	// Works only if whole image is displayed:
	// ctx.drawImage(img, sx, sy, sw, sh, dx, dy, dw, dh / vertSquashRatio);
	// The following works correct also when only a part of the image is displayed:
	ctx.drawImage(img, sx * vertSquashRatio, sy * vertSquashRatio,
	sw * vertSquashRatio, sh * vertSquashRatio,
	dx, dy, dw, dh);
}
```
