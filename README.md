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
