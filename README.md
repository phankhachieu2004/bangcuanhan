# banghoctiengtrung
bảng học tiếng trung cho Nhân
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trang web bảng trắng</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            display: flex;
            /* Thay đổi từ row sang column để các phần tử xếp dọc */
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f0f0f0;
            touch-action: none;
            /* Thêm padding trên để có không gian cho nút */
            padding-top: 20px;
        }

        .controls {
            margin-bottom: 10px; /* Khoảng cách giữa khu vực nút và bảng vẽ */
        }

        .controls button {
            margin: 0 5px;
            padding: 8px 15px;
            cursor: pointer;
            border: 1px solid #ccc;
            border-radius: 4px;
            background-color: #fff;
            font-size: 1rem;
        }

         .controls button:hover {
             background-color: #eee;
         }

        canvas {
            border: 1px solid #ccc;
            background-color: white;
            display: block;
            /* Kích thước cố định đã đặt trong thuộc tính HTML */
            touch-action: none; /* Ngăn cử chỉ cuộn/zoom trên canvas */
        }
    </style>
</head>
<body>

    <div class="controls">
        <button id="undoBtn">Quay lại</button>
        <button id="clearBtn">Xóa hết</button>
    </div>

    <canvas id="whiteboard" width="300" height="300"></canvas>

    <script>
        const canvas = document.getElementById('whiteboard');
        const context = canvas.getContext('2d');

        const undoBtn = document.getElementById('undoBtn');
        const clearBtn = document.getElementById('clearBtn');

        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;

        let drawingHistory = []; // Mảng lưu lịch sử các nét vẽ
        let currentStrokePoints = []; // Mảng tạm cho nét vẽ hiện tại

        // Kích thước canvas cố định trong HTML

        // Cài đặt các thuộc tính nét vẽ mặc định
        context.strokeStyle = '#000'; // Màu nét vẽ (đen)
        context.lineJoin = 'round'; // Kiểu nối các đoạn thẳng (bo tròn)
        context.lineCap = 'round'; // Kiểu kết thúc nét vẽ (bo tròn)
        context.lineWidth = 5; // Độ dày nét vẽ

        // --- Hàm trợ giúp: Vẽ lại toàn bộ lịch sử ---
        function redrawCanvas() {
            context.clearRect(0, 0, canvas.width, canvas.height); // Xóa canvas

            drawingHistory.forEach(stroke => {
                context.beginPath();
                if (stroke.length > 0) {
                    context.moveTo(stroke[0].x, stroke[0].y);
                    for (let i = 1; i < stroke.length; i++) {
                        context.lineTo(stroke[i].x, stroke[i].y);
                    }
                    context.stroke();
                }
            });
        }

        // --- Hàm lưu lịch sử vẽ vào LocalStorage ---
        function saveDrawing() {
            try {
                // Chuyển mảng drawingHistory thành chuỗi JSON và lưu vào LocalStorage
                const jsonHistory = JSON.stringify(drawingHistory);
                localStorage.setItem('whiteboardDrawing', jsonHistory);
                // console.log("Drawing saved."); // Debug
            } catch (e) {
                console.error("Could not save drawing:", e);
                // Xử lý lỗi nếu bộ nhớ đầy, vv.
            }
        }

        // --- Hàm tải lịch sử vẽ từ LocalStorage ---
        function loadDrawing() {
            try {
                // Lấy dữ liệu từ LocalStorage
                const savedData = localStorage.getItem('whiteboardDrawing');
                if (savedData) {
                    // Chuyển chuỗi JSON thành mảng và gán cho drawingHistory
                    drawingHistory = JSON.parse(savedData);
                     // console.log("Drawing loaded. History size:", drawingHistory.length); // Debug
                    // Vẽ lại canvas với dữ liệu đã tải
                    redrawCanvas();
                } else {
                    // Nếu không có dữ liệu lưu, khởi tạo mảng rỗng
                    drawingHistory = [];
                     // console.log("No saved drawing found."); // Debug
                }
            } catch (e) {
                console.error("Could not load drawing:", e);
                // Nếu lỗi khi tải/parse, đảm bảo lịch sử trống
                drawingHistory = [];
            }
        }


        // --- Các hàm xử lý CHUỘT (Thêm gọi hàm saveDrawing) ---
        function startDrawingMouse(e) {
            isDrawing = true;
            [lastX, lastY] = [e.offsetX, e.offsetY];
            currentStrokePoints = [{x: lastX, y: lastY}];
        }

        function drawMouse(e) {
            if (!isDrawing) return;
            const currentX = e.offsetX;
            const currentY = e.offsetY;
            context.beginPath();
            context.moveTo(lastX, lastY);
            context.lineTo(currentX, currentY);
            context.stroke();
            currentStrokePoints.push({x: currentX, y: currentY});
            [lastX, lastY] = [currentX, currentY];
        }

        function stopDrawingMouse() {
            if (!isDrawing) return;
            isDrawing = false;
            if (currentStrokePoints.length > 0) {
                drawingHistory.push(currentStrokePoints);
                saveDrawing(); // *** Lưu sau khi thêm nét vẽ ***
            }
            currentStrokePoints = [];
        }

        // --- Các hàm xử lý CHẠM (Thêm gọi hàm saveDrawing) ---
        function startDrawingTouch(e) {
            e.preventDefault();
            isDrawing = true;
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            lastX = touch.clientX - rect.left;
            lastY = touch.clientY - rect.top;
            currentStrokePoints = [{x: lastX, y: lastY}];
        }

        function drawTouch(e) {
            if (!isDrawing) return;
            e.preventDefault();
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            const currentX = touch.clientX - rect.left;
            const currentY = touch.clientY - rect.top;
            context.beginPath();
            context.moveTo(lastX, lastY);
            context.lineTo(currentX, currentY);
            context.stroke();
            currentStrokePoints.push({x: currentX, y: currentY});
            lastX = currentX;
            lastY = currentY;
        }

        function stopDrawingTouch() {
            if (!isDrawing) return;
            isDrawing = false;
             if (currentStrokePoints.length > 0) {
                drawingHistory.push(currentStrokePoints);
                saveDrawing(); // *** Lưu sau khi thêm nét vẽ ***
            }
            currentStrokePoints = [];
        }


        // --- Hàm xử lý nút "Quay lại" (Thêm gọi hàm saveDrawing) ---
        function undoLast() {
            if (drawingHistory.length > 0) {
                drawingHistory.pop(); // Xóa nét vẽ cuối cùng
                redrawCanvas(); // Vẽ lại
                saveDrawing(); // *** Lưu trạng thái mới sau khi Undo ***
            }
        }

        // --- Hàm xử lý nút "Xóa hết" (Thêm xóa khỏi LocalStorage) ---
        function clearCanvas() {
            context.clearRect(0, 0, canvas.width, canvas.height); // Xóa canvas
            drawingHistory = []; // Xóa lịch sử trong bộ nhớ
            currentStrokePoints = []; // Xóa điểm tạm
            localStorage.removeItem('whiteboardDrawing'); // *** Xóa dữ liệu khỏi LocalStorage ***
            // console.log("Đã xóa hết."); // Debug
        }


        // --- Gắn các bộ lắng nghe sự kiện ---

        // Sự kiện click cho các nút
        undoBtn.addEventListener('click', undoLast);
        clearBtn.addEventListener('click', clearCanvas);

        // Sự kiện Chuột (giữ nguyên)
        canvas.addEventListener('mousedown', startDrawingMouse);
        canvas.addEventListener('mousemove', drawMouse);
        canvas.addEventListener('mouseup', stopDrawingMouse);
        canvas.addEventListener('mouseout', stopDrawingMouse);

        // Sự kiện Chạm (giữ nguyên)
        canvas.addEventListener('touchstart', startDrawingTouch);
        canvas.addEventListener('touchmove', drawTouch);
        canvas.addEventListener('touchend', stopDrawingTouch);
        canvas.addEventListener('touchcancel', stopDrawingTouch);

        // --- TẢI DỮ LIỆU KHI TRANG WEB TẢI ---
        loadDrawing(); // Gọi hàm tải dữ liệu ngay khi script chạy

    </script>

</body>
</html>
