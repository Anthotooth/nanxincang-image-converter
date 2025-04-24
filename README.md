<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>南新仓风格图片转换器</title>
  <!-- 引入 Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- 引入 React 和 ReactDOM -->
  <script src="https://unpkg.com/react@17/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@17/umd/react-dom.production.min.js"></script>
  <!-- 引入 Babel -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.26.0/babel.min.js"></script>
</head>

<body class="bg-[#232323] text-white font-[PangMenZhengDao]">

  <div id="root"></div>

  <!-- React 代码 -->
  <script type="text/babel">
    const { useState, useRef } = React;

    function App() {
      const [imageURL, setImageURL] = useState(null);
      const canvasRef = useRef(null);

      // 上传图像
      const handleImageUpload = (e) => {
        const file = e.target.files[0];
        if (!file) {
          console.error('No file selected!');
          return;
        }

        const reader = new FileReader();
        reader.onload = (event) => {
          setImageURL(event.target.result);
        };
        reader.readAsDataURL(file);
      };

      // 颜色列表
      const colors = [
        { r: 77, g: 74, b: 71 },
        { r: 203, g: 174, b: 128 },
        { r: 132, g: 60, b: 43 },
        { r: 236, g: 225, b: 207 },
        { r: 41, g: 51, b: 63 },
        { r: 105, g: 123, b: 105 },
        { r: 109, g: 20, b: 25 },
        { r: 23, g: 23, b: 23 }
      ];

      // 计算两种颜色之间的距离（欧几里得距离）
      const colorDistance = (c1, c2) => {
        return Math.sqrt(
          Math.pow(c1.r - c2.r, 2) +
          Math.pow(c1.g - c2.g, 2) +
          Math.pow(c1.b - c2.b, 2)
        );
      };

      // 图像像素化并用颜色替代
      const processImage = () => {
        const canvas = canvasRef.current;
        const ctx = canvas.getContext("2d");
        const img = new Image();
        img.onload = () => {
          // 设置canvas大小为图像原始大小
          const imgWidth = img.width;
          const imgHeight = img.height;
          canvas.width = imgWidth;
          canvas.height = imgHeight;

          // 将图像绘制到canvas上
          ctx.drawImage(img, 0, 0, imgWidth, imgHeight);

          const blockSize = 16; // 每个像素块的大小
          // 遍历图像，进行像素化处理
          for (let y = 0; y < imgHeight; y += blockSize) {
            for (let x = 0; x < imgWidth; x += blockSize) {
              // 获取每个小块区域的像素数据
              const imageData = ctx.getImageData(x, y, blockSize, blockSize);
              const pixels = imageData.data;

              let r = 0, g = 0, b = 0;
              // 计算块内所有像素的平均颜色
              for (let i = 0; i < pixels.length; i += 4) {
                r += pixels[i];
                g += pixels[i + 1];
                b += pixels[i + 2];
              }

              r /= (pixels.length / 4);
              g /= (pixels.length / 4);
              b /= (pixels.length / 4);

              // 计算与每个颜色的欧几里得距离，选择最接近的颜色
              let closestColor = colors[0];
              let minDistance = colorDistance({ r, g, b }, closestColor);

              colors.forEach((color) => {
                const dist = colorDistance({ r, g, b }, color);
                if (dist < minDistance) {
                  closestColor = color;
                  minDistance = dist;
                }
              });

              // 使用最接近的颜色填充像素块
              ctx.fillStyle = `rgb(${closestColor.r}, ${closestColor.g}, ${closestColor.b})`;
              ctx.fillRect(x, y, blockSize, blockSize);
            }
          }
        };
        if (imageURL) img.src = imageURL;
      };

      // 下载处理后的图像
      const downloadImage = () => {
        const canvas = canvasRef.current;
        const link = document.createElement("a");

        // 获取canvas的图片数据URL
        const imageDataURL = canvas.toDataURL("image/png");

        // 设置下载链接
        link.href = imageDataURL;
        link.download = "pixelated-image-with-colors.png"; // 设置默认下载文件名

        // 触发点击事件进行下载
        link.click();
      };

      return (
        <div className="min-h-screen text-white p-6 flex flex-col items-center">
          <h1 className="text-4xl font-bold mb-4 text-[#6d1419] sm:text-3xl md:text-4xl">
            南新仓风格图片转换器
          </h1>

          <input
            type="file"
            accept="image/*"
            onChange={handleImageUpload}
            className="mb-4 px-4 py-2 border border-[#6d1419] rounded-lg sm:w-3/4 md:w-1/2 lg:w-1/3"
          />

          <button
            onClick={processImage}
            className="bg-[#6d1419] px-6 py-3 rounded-xl text-lg mb-6 hover:bg-[#4c0c13] transition-colors sm:w-3/4 md:w-1/2 lg:w-1/3"
          >
            转换图像
          </button>

          <canvas ref={canvasRef} className="border-[#6d1419] border-2 mb-6 w-full sm:w-3/4 md:w-1/2 lg:w-1/3" />

          <button
            onClick={downloadImage}
            className="bg-[#6d1419] px-6 py-3 rounded-xl text-lg hover:bg-[#4c0c13] transition-colors sm:w-3/4 md:w-1/2 lg:w-1/3"
          >
            下载图像
          </button>
        </div>
      );
    }

    // 渲染 React 应用
    ReactDOM.render(<App />, document.getElementById("root"));
  </script>
</body>

</html>
