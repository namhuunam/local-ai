# CÀI ĐẶT OLLAMA TRÊN WINDOWS
1 Vào https://ollama.ai/download/windows
2 Download file OllamaSetup.exe
3 Chạy installer với quyền Administrator
Kiểm tra GPU được detect:
```bash
ollama serve
```
Xóa model cũ:
```bash
ollama rm qwen:4b
```
Kiểm tra model nào đang có
```bash
ollama list
```
Cài đặt model:
```bash
ollama pull qwen2:7b-instruct
```
```bash
ollama pull mistral:7b-instruct
```
### 1 Download manual Windows:
1 Vào https://github.com/cloudflare/cloudflared/releases

2 Download cloudflared-windows-amd64.exe

3 Đổi tên thành cloudflared.exe

4 Copy vào C:\Windows\System32\ (cần quyền Admin)

### 1.1 Download manual ubuntu:
```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
```
```bash
sudo dpkg -i cloudflared-linux-amd64.deb
```

### 2 Từ Zero Trust Dashboard:

1 Vào https://one.dash.cloudflare.com
2 Chọn Networks → Tunnels

3 Hoặc Networks → Cloudflare Tunnel

### 3 Tạo tunnel mới:

1 Click Create a tunnel

2 Chọn Cloudflared

3 Đặt tên tunnel: qwen-ai-tunnel

4 Click Save tunnel

### BƯỚC 34: CÀI ĐẶT CONNECTOR

3.1 Copy install command:

Trong tunnel dashboard sẽ có lệnh như:

```bash
cloudflared.exe service install eyJhIjoiYWJjMTIzLi4uIn0=
```

3.2 Chạy lệnh với quyền Administrator:

```bash
cloudflared.exe service install eyJhIjoiYWJjMTIzLi4uIn0=
```

Hoặc nếu chưa có cloudflared:

```bash
# Download và install
winget install --id Cloudflare.cloudflared

# Rồi chạy lệnh install
cloudflared.exe service install eyJhIjoiYWJjMTIzLi4uIn0=
```

### BƯỚC 5: CẤU HÌNH PUBLIC HOSTNAME

4.1 Trong Tunnel Dashboard:

1 Chọn tab Public Hostnames

2 Click Add a public hostname

4.2 Cấu hình hostname:

 Subdomain: ai (hoặc qwen, ollama)
 
 Domain: codepion.xyz
 
 Path: để trống
 
 Service Type: HTTP
 
 URL: localhost:11434
 
Kết quả: ai.codepion.xyz → localhost:11434
HTTP Settings
HTTP Host Header:localhost
4.3 Click Save hostname

### BƯỚC 6: TEST DOMAIN

6.1 Đảm bảo Ollama đang chạy:

```bash
curl http://localhost:11434/api/tags
curl https://ai.codepion.xyz/api/tags
```

Auto Start Cloudflare Tunnel

```bash
cloudflared service install
```

```bash
sc config cloudflared start= auto
```

### Lệnh kiểm tra ollama có sử dụng GPU trên WSL2 không
```bash
watch -n 0.5 nvidia-smi
```


### tạo enpoint riêng cho cpu và gpu
Bước 1: Tạo cấu trúc thư mục mới
```bash
# Tạo thư mục config riêng cho mỗi instance
mkdir -p /home/ubuntu/.ollama-cpu
mkdir -p /home/ubuntu/.ollama-gpu

# Tạo symlink để share models (tiết kiệm dung lượng)
ln -s /home/ubuntu/.ollama/models /home/ubuntu/.ollama-cpu/models
ln -s /home/ubuntu/.ollama/models /home/ubuntu/.ollama-gpu/models

# Kiểm tra symlink đã tạo đúng chưa
ls -la /home/ubuntu/.ollama-cpu/
ls -la /home/ubuntu/.ollama-gpu/
```

```bash
nano ~/ollama-cpu.sh
```
```bash
#!/bin/bash
export CUDA_VISIBLE_DEVICES=""
export OLLAMA_HOST="127.0.0.1:11436"
export OLLAMA_MODELS="/home/ubuntu/.ollama-cpu"

echo "🖥️ Starting Ollama CPU instance on port 11436..."
echo "Models path: $OLLAMA_MODELS"
echo "Shared models via symlink"
ollama serve &
wait
```

```bash
nano ~/ollama-gpu.sh
```
```bash
#!/bin/bash
export CUDA_VISIBLE_DEVICES="0"
export OLLAMA_HOST="127.0.0.1:11435"
export OLLAMA_MODELS="/home/ubuntu/.ollama-gpu"

echo "🚀 Starting Ollama GPU instance on port 11435..."
echo "Models path: $OLLAMA_MODELS"
echo "Shared models via symlink"
ollama serve &
wait
```
```bash
# Pull cho CPU instance
OLLAMA_HOST="127.0.0.1:11436" ollama pull llama3:8b

# Pull cho GPU instance  
OLLAMA_HOST="127.0.0.1:11435" ollama pull qwen2:7b-instruct
```
Bước 4: Khởi động và test
```bash
./ollama-cpu.sh
```
```bash
./ollama-gpu.sh
```
Bước 5: Tự Khởi động
Tạo systemd service cho CPU
```bash
sudo nano /etc/systemd/system/ollama-cpu.service
```
```bash
[Unit]
Description=Ollama CPU Instance (llama3:8b)
After=network.target
Wants=network.target

[Service]
Type=simple
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu
ExecStart=/home/ubuntu/ollama-cpu.sh
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```
Tạo systemd service cho GPU
```bash
sudo nano /etc/systemd/system/ollama-gpu.service
```
```bash
[Unit]
Description=Ollama GPU Instance (qwen2:7b-instruct)
After=network.target ollama-cpu.service
Wants=network.target

[Service]
Type=simple
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu
ExecStart=/home/ubuntu/ollama-gpu.sh
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```
Cấp quyền thực thi cho scripts
```bash
chmod +x ~/ollama-cpu.sh
chmod +x ~/ollama-gpu.sh
```
Kích hoạt services
```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable auto-start khi boot
sudo systemctl enable ollama-cpu
sudo systemctl enable ollama-gpu

# Start services ngay
sudo systemctl start ollama-cpu
sudo systemctl start ollama-gpu

# Kiểm tra status
sudo systemctl status ollama-cpu
sudo systemctl status ollama-gpu
```
