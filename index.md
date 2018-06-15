### Javascript

Resize image by canvas

```javascript
function resizeImage(file, callback) {
  if(!!file){
    console.log(file);
    var fileReader = new FileReader();
    fileReader.onload = function () {
      var img = new Image(), degrees = 0;
      img.onload = function () {
        var canvas = document.createElement("canvas"),
            imgWidth = img.width,
            imgHeight = img.height,
            ctx = canvas.getContext("2d"),
            rota = imgWidth - imgHeight,
            newW = (imgWidth < 1500) ? imgWidth : 1500,
            newH = (imgHeight < 1500) ? imgHeight : 1500;
        // file size > 2MB
        if(file.size > 1024 * 1024 * 2){
          // width < height
          if(rota < 0){
            imgWidth = newW;
            imgHeight = newH;
          }else if(rota > 0 && rota !== 1){ // width > height 
            imgWidth = newW;
            imgHeight = newH;
          }else{ // width = height
            imgWidth = newW;
            imgHeight = newH;
          }
        }

        canvas.width = newW;
        canvas.height = newH;
        ctx.clearRect(0,0,canvas.width,canvas.height);
        ctx.drawImage(img, 0, 0, newW, newH);

        // rotate image
        rotateImg(file, canvas.toDataURL(file.type, 1.0), function(dataURL){
           callback(dataURLtoFile(dataURL, file.name), dataURL);
        });

      };
      img.src = this.result;
    }
    fileReader.readAsDataURL(file);
  }
}

function dataURLtoFile(dataURL, filename) {
  var arr = dataURL.split(','), mime = arr[0].match(/:(.*?);/)[1],
      bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
  while(n--){
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new File([u8arr], filename, {type:mime});
}

function rotateImg(file, dataURL, callback) {
  getOrientation(file, function(rotate){
    var degrees = 0,
      canvas = document.createElement("canvas"),
      ctx = canvas.getContext("2d"),
      img = document.createElement("img");
    
    switch (rotate) {
      case 1: break;
      case 2: break;
      case 3: degrees = 180; break;
      case 4: break;
      case 5: degrees = 90; break;
      case 6: degrees = 90; break;
      case 7: break;
      case 8: degrees = 270; break;
    }
    
    img.onload = function () {
      canvas.width = img.width;
      canvas.height = img.height;
      ctx.drawImage(img, 0, 0);

      if (degrees === 0 || degrees === 180) {
        canvas.width = img.width;
        canvas.height = img.height;
      } else {
        // swap
        canvas.width = img.height;
        canvas.height = img.width;
      }
      ctx.save();
      // rotate around center of canvas
      ctx.translate(canvas.width / 2, canvas.height / 2);
      ctx.rotate(degrees * Math.PI / 180);
      ctx.drawImage(img, -img.width * 0.5, -img.height * 0.5);
      ctx.restore();
      callback(canvas.toDataURL(file.type, 1.0));
    }
    img.src = dataURL;
  });
}

function getOrientation(file, callback) {
  var reader = new FileReader();
  reader.onload = function (e) {
    var view = new DataView(e.target.result),
      length = view.byteLength,
      offset = 2;
    if (view.getUint16(0, false) != 0xFFD8)
      return callback(-2);

    while (offset < length) {
      var marker = view.getUint16(offset, false);
      offset += 2;
      if (marker == 0xFFE1) {
        if (view.getUint32(offset += 2, false) != 0x45786966)
          return callback(-1);

        var little = view.getUint16(offset += 6, false) == 0x4949;
        offset += view.getUint32(offset + 4, little);
        var tags = view.getUint16(offset, little);
        offset += 2;
        for (var i = 0; i < tags; i++)
          if (view.getUint16(offset + (i * 12), little) == 0x0112)
            return callback(view.getUint16(offset + (i * 12) + 8, little));
      } else if ((marker & 0xFF00) != 0xFF00) break;
      else offset += view.getUint16(offset, false);
    }
    return callback(-1);
  };
  reader.readAsArrayBuffer(file);
}
```

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
