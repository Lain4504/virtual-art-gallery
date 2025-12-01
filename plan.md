# Kế hoạch thay đổi API và sử dụng Local Images

## Tổng quan

Dự án này hỗ trợ nhiều nguồn dữ liệu tranh (API) khác nhau. Bạn có thể:
- Thay đổi API mặc định
- Thêm API mới
- Sử dụng file JSON và ảnh local thay vì gọi API

---

## 📋 Mục lục

1. [Cấu trúc API hiện tại](#cấu-trúc-api-hiện-tại)
2. [Cách chọn API](#cách-chọn-api)
3. [Thay đổi API mặc định](#thay-đổi-api-mặc-định)
4. [Sử dụng Local Images (đã có sẵn)](#sử-dụng-local-images-đã-có-sẵn)
5. [Thêm API mới](#thêm-api-mới)
6. [Chuẩn bị dữ liệu Local](#chuẩn-bị-dữ-liệu-local)
7. [Các bước thực hiện cụ thể](#các-bước-thực-hiện-cụ-thể)

---

## 🔍 Cấu trúc API hiện tại

### File cấu hình API
**Vị trí:** `api/api.js`

```javascript
module.exports = {
    default: "artic",        // API mặc định
    artic: require("./artic"), // API từ Art Institute of Chicago
    local: require("./local")  // API từ file JSON local
};
```

### Cấu trúc một API module
Mỗi API phải export 2 functions:

1. **`fetchList(from, count)`**: Lấy danh sách tranh
   - `from`: Vị trí bắt đầu (index)
   - `count`: Số lượng tranh cần lấy
   - Return: Array các object chứa metadata tranh

2. **`fetchImage(obj, advicedResolution)`**: Lấy ảnh tranh
   - `obj`: Object metadata từ fetchList
   - `advicedResolution`: "low", "mid", hoặc "high"
   - Return: `{title: string, image: Blob}`

---

## 🎯 Cách chọn API

### Cách 1: Qua URL parameter (không cần thay đổi code)
```
http://localhost:9966/?api=local
http://localhost:9966/?api=artic
```

### Cách 2: Thay đổi default trong code
Sửa file `api/api.js`:
```javascript
module.exports = {
    default: "local",  // Thay "artic" thành "local"
    artic: require("./artic"),
    local: require("./local")
};
```

---

## 📁 Sử dụng Local Images (đã có sẵn)

### Cấu trúc thư mục hiện tại
```
images/
├── images.json      # File JSON chứa metadata
├── generateList.js  # Script tự động tạo images.json
├── 1.png
├── 2.png
├── 3.png
└── ...
```

### File JSON format
**Vị trí:** `images/images.json`

```json
{
    "images": [
        {
            "title": "Tên tranh 1",
            "file": "1.png"
        },
        {
            "title": "Tên tranh 2",
            "file": "2.png"
        }
    ]
}
```

### Local API đã implement
**Vị trí:** `api/local.js`

API này đã sẵn sàng sử dụng, đọc từ `images/images.json` và load ảnh từ thư mục `images/`.

---

## 🆕 Thêm API mới

### Bước 1: Tạo file API mới
Tạo file mới trong thư mục `api/`, ví dụ: `api/custom.js`

```javascript
'strict mode';

// Import file JSON nếu cần
const images = require("../images/custom.json").images.map((img, i) => ({
    ...img, 
    image_id: i
}));

module.exports = {
    fetchList: async function (from, count) {
        // Trả về danh sách tranh từ vị trí 'from', số lượng 'count'
        return images.slice(from, from + count);
    },
    
    fetchImage: async function (obj, advicedResolution) {
        // obj chứa metadata từ fetchList
        // advicedResolution: "low", "mid", "high"
        
        // Đường dẫn đến ảnh
        const url = "images/custom/" + obj.file;
        const blob = await fetch(url).then(res => res.blob());
        
        return {
            title: obj.title,
            image: blob
        };
    }
};
```

### Bước 2: Đăng ký API mới
Sửa file `api/api.js`:

```javascript
module.exports = {
    default: "custom",  // Hoặc giữ nguyên "artic"/"local"
    artic: require("./artic"),
    local: require("./local"),
    custom: require("./custom")  // Thêm dòng này
};
```

### Bước 3: Sử dụng
```
http://localhost:9966/?api=custom
```

---

## 📦 Chuẩn bị dữ liệu Local

### Cách 1: Tự động tạo images.json (Khuyến nghị)

**Script có sẵn:** `images/generateList.js`

Script này tự động quét thư mục `images/` và tạo file `images.json`.

**Cách dùng:**
```bash
npm run genList
```

**Lưu ý:** Script sẽ tự động chạy khi bạn chạy `npm start` hoặc `npm run build` (xem `package.json`).

### Cách 2: Tạo thủ công

1. **Chuẩn bị ảnh:**
   - Đặt tất cả ảnh vào thư mục `images/`
   - Format: PNG, JPG, JPEG đều được
   - Tên file: Bất kỳ (ví dụ: `mona-lisa.jpg`, `starry-night.png`)

2. **Tạo file JSON:**
   - Tạo file `images/images.json`
   - Format như ví dụ ở trên
   - Mỗi object cần có:
     - `title`: Tên hiển thị của tranh
     - `file`: Tên file ảnh (bao gồm extension)

### Cách 3: Sử dụng thư mục con

Nếu bạn muốn tổ chức ảnh trong thư mục con:

**Ví dụ cấu trúc:**
```
images/
├── images.json
├── paintings/
│   ├── painting1.jpg
│   ├── painting2.jpg
│   └── ...
└── sculptures/
    ├── sculpture1.jpg
    └── ...
```

**File JSON:**
```json
{
    "images": [
        {
            "title": "Painting 1",
            "file": "paintings/painting1.jpg"
        },
        {
            "title": "Sculpture 1",
            "file": "sculptures/sculpture1.jpg"
        }
    ]
}
```

**Sửa `api/local.js`:**
```javascript
fetchImage: async function (obj, advicedResolution) {
    const url = "images/" + obj.file;  // Đã đúng, không cần sửa
    // ...
}
```

---

## 🔧 Các bước thực hiện cụ thể

### Kịch bản 1: Chuyển sang dùng Local Images (API local đã có)

#### Bước 1: Chuẩn bị ảnh
```bash
# Đặt tất cả ảnh vào thư mục images/
# Ví dụ:
images/
├── mona-lisa.jpg
├── starry-night.png
├── the-scream.jpg
└── ...
```

#### Bước 2: Tạo danh sách tự động
```bash
npm run genList
```

Hoặc tạo thủ công file `images/images.json` với format đúng.

#### Bước 3: Thay đổi API mặc định (Tùy chọn)
Sửa `api/api.js`:
```javascript
module.exports = {
    default: "local",  // Thay đổi từ "artic" sang "local"
    artic: require("./artic"),
    local: require("./local")
};
```

#### Bước 4: Test
```bash
npm start
# Mở browser: http://localhost:9966
# Hoặc nếu chưa đổi default: http://localhost:9966/?api=local
```

---

### Kịch bản 2: Thêm API mới với JSON riêng

#### Bước 1: Tạo file JSON mới
Tạo `images/my-gallery.json`:
```json
{
    "images": [
        {
            "title": "Tác phẩm 1",
            "file": "artwork1.jpg",
            "artist": "Họa sĩ A"
        },
        {
            "title": "Tác phẩm 2",
            "file": "artwork2.png",
            "artist": "Họa sĩ B"
        }
    ]
}
```

#### Bước 2: Tạo thư mục ảnh
```bash
mkdir images/my-gallery
# Copy ảnh vào: images/my-gallery/artwork1.jpg, artwork2.png, ...
```

#### Bước 3: Tạo API module
Tạo `api/my-gallery.js`:
```javascript
'strict mode';

const images = require("../images/my-gallery.json").images.map((img, i) => ({
    ...img, 
    image_id: i
}));

module.exports = {
    fetchList: async function (from, count) {
        return images.slice(from, from + count);
    },
    
    fetchImage: async function (obj, advicedResolution) {
        const url = "images/my-gallery/" + obj.file;
        const blob = await fetch(url).then(res => res.blob());
        return {
            title: obj.artist ? `${obj.title} - ${obj.artist}` : obj.title,
            image: blob
        };
    }
};
```

#### Bước 4: Đăng ký API
Sửa `api/api.js`:
```javascript
module.exports = {
    default: "my-gallery",
    artic: require("./artic"),
    local: require("./local"),
    "my-gallery": require("./my-gallery")
};
```

#### Bước 5: Test
```bash
npm start
# http://localhost:9966/?api=my-gallery
```

---

### Kịch bản 3: Tắt API, chỉ dùng Local

#### Bước 1: Đổi default
Sửa `api/api.js`:
```javascript
module.exports = {
    default: "local",  // Chỉ dùng local
    local: require("./local")
    // Có thể xóa artic nếu không cần
};
```

#### Bước 2: Chuẩn bị ảnh và JSON
- Đặt ảnh vào `images/`
- Chạy `npm run genList` hoặc tạo `images.json` thủ công

#### Bước 3: Build và chạy
```bash
npm run build
# Hoặc
npm start
```

---

## 📝 Lưu ý quan trọng

### 1. Format ảnh
- Hỗ trợ: PNG, JPG, JPEG
- Khuyến nghị: JPG cho ảnh ảnh thật, PNG cho ảnh có transparency
- Kích thước: Không giới hạn, nhưng nên tối ưu để tải nhanh

### 2. Đường dẫn ảnh
- Trong `images.json`: `file` phải là đường dẫn tương đối từ thư mục `images/`
- Ví dụ: Nếu ảnh ở `images/paintings/art.jpg`, thì `file: "paintings/art.jpg"`

### 3. Resolution trong Local API
- Local API hiện tại **không hỗ trợ dynamic resolution** như ARTIC API
- Tất cả ảnh sẽ được load với độ phân giải gốc
- Nếu muốn hỗ trợ resolution, cần:
  - Tạo nhiều version (low/mid/high) của mỗi ảnh
  - Sửa `api/local.js` để chọn ảnh theo resolution

### 4. Cache và Performance
- Ảnh được cache trong memory
- Texture được reuse khi unload
- Không cần lo về rate limit như API

### 5. Build process
- Script `genList` tự động chạy trước `start` và `build`
- Nếu thêm ảnh mới, cần:
  - Chạy lại `npm run genList`
  - Hoặc restart dev server (nó sẽ tự chạy)

---

## 🎨 Ví dụ hoàn chỉnh

### Setup Local Gallery từ đầu

1. **Tạo thư mục và thêm ảnh:**
```bash
# Đảm bảo bạn đang ở thư mục gốc dự án
cd images/
# Copy ảnh vào đây
cp ~/my-paintings/*.jpg .
```

2. **Tạo images.json:**
```bash
npm run genList
```

3. **Sửa default API:**
Sửa `api/api.js`:
```javascript
module.exports = {
    default: "local",
    artic: require("./artic"),
    local: require("./local")
};
```

4. **Chạy:**
```bash
npm start
# Mở: http://localhost:9966
```

---

## 🔄 Migration từ API sang Local

Nếu bạn đang dùng ARTIC API và muốn chuyển sang local:

1. **Export dữ liệu từ API** (nếu cần):
   - Có thể viết script tải ảnh từ ARTIC API
   - Lưu metadata vào JSON

2. **Tải ảnh về local:**
   - Lưu vào thư mục `images/`

3. **Tạo JSON:**
   - Format như mẫu ở trên
   - Đảm bảo `file` trỏ đúng tên file

4. **Đổi default:**
   - Sửa `api/api.js` như hướng dẫn

5. **Test:**
   - Chạy và kiểm tra

---

## 📚 Tài liệu tham khảo

- File API mẫu: `api/local.js` (đơn giản nhất)
- File API phức tạp: `api/artic.js` (có dynamic resolution)
- Router: `api/api.js`
- Loader: `src/image.js`

---

## ❓ Troubleshooting

### Lỗi: "Cannot find module '../images/images.json'"
- **Nguyên nhân:** File JSON chưa được tạo
- **Giải pháp:** Chạy `npm run genList`

### Ảnh không hiển thị
- Kiểm tra đường dẫn trong `images.json` có đúng không
- Kiểm tra tên file có khớp không (case-sensitive)
- Kiểm tra console browser để xem lỗi fetch

### API không được nhận diện
- Kiểm tra đã đăng ký trong `api/api.js` chưa
- Kiểm tra tên API trong URL có đúng không
- Kiểm tra file API có export đúng format chưa

---

**Tạo bởi:** AI Assistant  
**Ngày:** 2024  
**Version:** 1.0

