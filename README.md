# Nhận diện vật thể trên Raspberry Pi 5 với Google Coral USB

Repo này cung cấp hướng dẫn đầy đủ cùng toàn bộ script cần thiết để chạy và huấn luyện mô hình nhận diện vật thể tùy chỉnh trên Raspberry Pi 5, dùng Google Coral USB Accelerator và TensorFlow Lite.

Video hướng dẫn: https://youtu.be/AE6fcQHJ_lE

---

## Ghi công

Dự án này dựa trên hướng dẫn gốc tại: https://www.youtube.com/watch?v=fVmAeK-GLXA&t=837s

Các lệnh và script đã được chỉnh sửa để chạy đúng trên Raspberry Pi 5.

---

## Tổng quan

Dự án hướng dẫn bạn qua các bước:
- Cài Python 3.9.12 trên Raspberry Pi 5 bằng pyenv
- Cài đặt và cấu hình Google Coral USB EdgeTPU
- Chạy nhận diện vật thể thời gian thực với TensorFlow Lite
- Gán nhãn ảnh tùy chỉnh bằng LabelImg
- Huấn luyện mô hình EfficientDet Lite tùy chỉnh trên Google Colab
- Triển khai mô hình đã huấn luyện ngược lại lên Raspberry Pi 5

---

## Yêu cầu

**Phần cứng:**
- Raspberry Pi 5
- Google Coral USB Accelerator
- Camera USB

**Phần mềm:**
- Raspberry Pi OS (khuyến nghị bản 64-bit)
- Python 3.9.12 (qua pyenv)
- Google Coral EdgeTPU runtime
- TensorFlow Lite runtime
- LabelImg (để gán nhãn)

---

## Cấu trúc repo

```
.
├── detect.py                          # Script nhận diện chính (chạy suy luận trên camera)
├── train.py                           # Script huấn luyện dùng cho Google Colab
├── test.py                            # Chép metadata từ best.tflite sang best_edgetpu.tflite
├── labelImg.py                        # File nguồn LabelImg đã vá
├── canvas.py                          # File nguồn canvas đã vá cho LabelImg
├── labelimg.sh                        # Script shell để cài và vá LabelImg
├── tflite_custom_model_edgetpu.ipynb  # Notebook Google Colab để huấn luyện
├── tensorflow-lite-bullseye-main.zip  # Gói cài đặt TensorFlow Lite
└── rpi5.txt                           # Ghi chú bổ sung khi cài đặt trên Raspberry Pi 5
```

---

## Bước 1: Cài Python 3.9.12 bằng pyenv

Mở terminal trên Raspberry Pi và chạy từng lệnh một.

**1.1 Cập nhật và nâng cấp hệ thống:**

```bash
sudo apt-get update
```

```bash
sudo apt-get upgrade
```

**1.2 Tạo thư mục dự án và vào thư mục đó:**

```bash
mkdir freedomtech
```

```bash
cd freedomtech
```

**1.3 Cài pyenv:**

```bash
curl https://pyenv.run | bash
```

**1.4 Thêm pyenv vào cấu hình shell (chạy từng dòng riêng):**

```bash
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
```

```bash
echo 'eval "$(pyenv init --path)"' >> ~/.bashrc
```

```bash
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
```

```bash
exec "$SHELL"
```

**1.5 Cài các gói phụ thuộc cần cho quá trình build:**

```bash
sudo apt-get install --yes libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev llvm libncurses-dev xz-utils tk-dev libgdbm-dev lzma tcl-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev wget curl make build-essential openssl
```

**1.6 Cài Python 3.9.12:**

```bash
pyenv install 3.9.12
```

**1.7 Đặt Python 3.9.12 làm phiên bản local:**

```bash
pyenv local 3.9.12
```

**1.8 Kiểm tra lại phiên bản Python:**

```bash
python --version
```

---

## Bước 2: Cài Google Coral USB EdgeTPU

**2.1 Tạo môi trường ảo Python:**

```bash
python3 -m venv .venv
```

**2.2 Kích hoạt môi trường ảo:**

```bash
source .venv/bin/activate
```

**2.3 Tạo thư mục keyrings:**

```bash
sudo mkdir -p /etc/apt/keyrings
```

**2.4 Tải và thêm khóa GPG của Coral:**

```bash
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/coral-edgetpu.gpg
```

**2.5 Thêm repository gói của Coral:**

```bash
echo "deb [signed-by=/etc/apt/keyrings/coral-edgetpu.gpg] https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
```

**2.6 Cập nhật danh sách gói:**

```bash
sudo apt-get update
```

**2.7 Cài EdgeTPU runtime:**

```bash
sudo apt-get install libedgetpu1-std
```

Khởi động lại Raspberry Pi sau khi cài xong.

**2.8 Sau khi khởi động lại, vào lại thư mục dự án và kích hoạt môi trường ảo:**

```bash
cd freedomtech
```

```bash
source .venv/bin/activate
```

---

## Bước 3: Cài TensorFlow Lite

Giải nén `tensorflow-lite-bullseye-main.zip` vào thư mục `freedomtech`, rồi chạy:

```bash
cd tensorflow-lite-bullseye-main
chmod 775 tensorflow-lite.sh
bash tensorflow-lite.sh
```

Script `tensorflow-lite.sh` sẽ:
- Nâng cấp pip
- Cài tflite-runtime
- Clone repository TensorFlow examples

Sau khi script chạy xong, chuyển tới ví dụ object detection và chạy phần setup:

```bash
cd examples/lite/examples/object_detection/raspberry_pi/
bash setup.sh
```

Kiểm tra phiên bản OpenCV đã cài và cập nhật `requirements.txt` nếu cần, trước khi chạy `setup.sh`.

Chép `detect.py` từ repo này vào thư mục `freedomtech`. Đảm bảo phiên bản Python ghi trong `detect.py` khớp với 3.9.12.

Chạy thử nhanh với mô hình dựng sẵn:

```bash
python detect.py --model efficientdet_lite0_edgetpu.tflite --enableEdgeTPU
```

---

## Bước 4: Cài và cấu hình LabelImg

Mở một terminal mới KHÔNG ở trong môi trường ảo, rồi chạy:

```bash
sudo rm /usr/lib/python3.13/EXTERNALLY-MANAGED

cd Pi5_object_detection_with_coral-main/
```

Kiểm tra phiên bản Python ghi trong `labelimg.sh` và sửa cho khớp với hệ thống của bạn, sau đó cài:

```bash
chmod 775 labelimg.sh
bash labelimg.sh
```

Nếu sau khi mở LabelImg bạn thấy cảnh báo:

```
QStandardPaths: wrong permissions on runtime directory
```

Khắc phục bằng:

```bash
chmod 0700 /run/user/1000
```

Mở LabelImg:

```bash
labelImg
```

---

## Bước 5: Chụp ảnh huấn luyện bằng img.py

Trước khi gán nhãn, bạn cần có một bộ ảnh dữ liệu. Dùng `img.py` để chụp ảnh từ camera trực tiếp trên Raspberry Pi.

**5.1 Tạo thư mục images:**

```bash
mkdir images
```

**5.2 Mở `img.py` và cập nhật đường dẫn lưu ảnh** trỏ tới thư mục `images` của bạn. Tìm dòng sau và thay đường dẫn:

```python
cv2.imwrite("/home/pi/Downloads/yolov8-custom-object-detection-googlecoralusb-main/images/arduino_uno_%d.jpg" %cpt, frame)
```

Sửa lại cho khớp đường dẫn thật của thư mục `images`, ví dụ:

```python
cv2.imwrite("/home/pi/Pi5_object_detection_with_coral-main/Images/object_%d.jpg" %cpt, frame)
```

Bạn cũng có thể đổi tiền tố tên file (`object_`) thành tên mô tả vật thể của bạn.

**5.3 Cài opencv-python (dùng Python 3.13 của hệ thống, bên ngoài môi trường ảo):**

```bash
pip install opencv-python
```

**5.4 Mở img.py trong Thonny và chạy**

Trong lúc script chạy, hãy di chuyển vật thể trước camera:
- Đưa qua trái, qua phải
- Đưa lại gần, ra xa
- Xoay vật để lấy nhiều góc khác nhau
- Thay đổi phông nền và ánh sáng nếu có thể

Mặc định script sẽ chụp 30 khung hình và lưu vào thư mục `images`. Nếu muốn nhiều ảnh hơn thì sửa trong code.

---

## Bước 6: Gán nhãn ảnh

1. Mở LabelImg.
2. Bấm **Open Dir** và trỏ tới thư mục `images` của bạn.
3. Bấm **Change Save Dir** và đặt về cùng thư mục `images` để file nhãn được lưu cạnh ảnh.
4. Vẽ khung bao quanh vật thể và gán nhãn lớp.
5. Lưu nhãn ở định dạng Pascal VOC (XML).

Sau khi gán nhãn xong, sắp xếp lại bộ dữ liệu:

```
freedomtech/
├── train/
│   ├── image1.jpg
│   ├── image1.xml
│   └── ...
└── validate/
    ├── image2.jpg
    ├── image2.xml
    └── ...
```

---

## Bước 7: Huấn luyện mô hình trên Google Colab

Nén thư mục dữ liệu:

```bash
sudo zip -r freedomtech.zip freedomtech/*
```

Tải `freedomtech.zip` lên Google Drive.

Mở `tflite_custom_model_edgetpu.ipynb` trong Google Colab:
- Vào Runtime > Change runtime type
- Đặt Hardware accelerator là GPU (T4)
- Chạy lần lượt từng cell

Trước khi chạy cell 8, hãy tải `train.py` lên phiên Colab. Mở `train.py` và cập nhật tên các lớp cho khớp với nhãn bạn đã gán:

```python
train_data = object_detector.DataLoader.from_pascal_voc(
    'freedomtech/train',
    'freedomtech/train',
    ['your_class_1', 'your_class_2']   # Thay bằng tên lớp thực tế của bạn
)

val_data = object_detector.DataLoader.from_pascal_voc(
    'freedomtech/validate',
    'freedomtech/validate',
    ['your_class_1', 'your_class_2']   # Thay bằng tên lớp thực tế của bạn
)
```

Script huấn luyện dùng `EfficientDet Lite0` với cấu hình mặc định:
- Batch size: 4
- Epochs: 100
- Fine-tune toàn bộ mô hình: bật

Sau khi huấn luyện xong, file kết quả `best.tflite` sẽ được xuất ra.

Trước khi chạy cell 12, hãy tải `test.py` lên phiên Colab. Script này chép metadata từ `best.tflite` sang `best_edgetpu.tflite` để mô hình tương thích với trình biên dịch Coral EdgeTPU.

Nếu gặp lỗi hoặc cần sửa code, luôn reset môi trường hoàn toàn:
1. Edit > Clear all outputs
2. Runtime > Disconnect and delete runtime
3. Khởi động lại phiên

---

## Bước 8: Triển khai mô hình đã huấn luyện lên Raspberry Pi 5

Tải `best_edgetpu.tflite` từ Google Colab về và chép vào thư mục `freedomtech` trên Raspberry Pi.

Kích hoạt môi trường ảo và chạy nhận diện vật thể với mô hình tùy chỉnh của bạn:

```bash
source .venv/bin/activate
python detect.py --model best_edgetpu.tflite --enableEdgeTPU
```

---

## Ghi chú

- Luôn nhớ đổi tên nhãn lớp trong `train.py` cho khớp bộ dữ liệu của bạn trước khi huấn luyện.
- Script `labelimg.sh` phải dùng đúng phiên bản Python khớp với bản cài trên hệ thống.
- Khuyến nghị dùng `libedgetpu1-std` (tốc độ tiêu chuẩn) để hoạt động ổn định. Gói `libedgetpu1-max` chạy TPU ở xung nhịp tối đa nhưng có thể làm thiết bị nóng.
- Script `test.py` yêu cầu cả `best.tflite` và `best_edgetpu.tflite` phải nằm cùng thư mục trước khi chạy.

---

## Giấy phép

Dự án này dựa trên repository TensorFlow Examples, cấp phép theo Apache License 2.0.
