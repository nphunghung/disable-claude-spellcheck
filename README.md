# Tắt gạch đỏ kiểm tra chính tả trong Claude Desktop

Claude desktop bị gạch đỏ kiểm tra chính tả rất khó chịu. Đã tìm và thử nhiều cách được chia sẻ trên mạng nhưng không được. Mình đã hỏi chính claude và được hướng dẫn chạy Script thay đổi lại file cấu hình của Claude desktop. Lệnh chạy trên Windows 11, chưa thử trên các hệ điều hành khác.

## Cách dùng

1. Thoát hẳn Claude. Chuột phải biểu tượng Claude ở khay hệ thống gần đồng hồ và chọn Quit. Sau đó mở Task Manager (Ctrl + Shift + Esc) và kiểm tra không còn tiến trình Claude nào chạy.
2. Nhấn phím Windows, gõ PowerShell, rồi mở Windows PowerShell (không cần quyền Admin).
3. Chạy script này để tìm file cấu hình:
   ```powershell
   Get-ChildItem "$env:APPDATA", "$env:LOCALAPPDATA" -Recurse -Filter "Preferences" -File -ErrorAction SilentlyContinue | Where-Object { $_.FullName -match "Claude" } | Select-Object FullName, LastWriteTime
   ```
   Lệnh này có thể chạy mất khoảng 1 đến 2 phút. Nó chỉ tìm và liệt kê, không sửa gì.
4. Nếu lệnh trên không ra kết quả, thử chạy thêm lệnh này:
   ```powershell
   Get-ChildItem "$env:APPDATA", "$env:LOCALAPPDATA", "$env:LOCALAPPDATA\Packages" -Directory -ErrorAction SilentlyContinue | Where-Object { $_.Name -match "Claude|Anthropic" } | Select-Object FullName
   ```
5. Thoát hẳn Claude. Chuột phải biểu tượng ở khay hệ thống và chọn Quit, rồi kiểm tra trong Task Manager không còn tiến trình Claude nào. Bước này bắt buộc, vì nếu Claude còn chạy nó sẽ ghi đè lại file.
6. Dán đoạn này vào PowerShell và nhấn Enter:
   ```powershell
    $files = Get-ChildItem "$env:LOCALAPPDATA\Claude-Data" -Recurse -Filter "Preferences" -File -ErrorAction SilentlyContinue
    foreach ($f in $files) {
      $p = $f.FullName
      Copy-Item $p "$p.bak" -Force
      $j = Get-Content $p -Raw -Encoding UTF8 | ConvertFrom-Json
      if (-not $j.browser) { $j | Add-Member -NotePropertyName browser -NotePropertyValue ([pscustomobject]@{}) }
      $j.browser | Add-Member -NotePropertyName enable_spellchecking -NotePropertyValue $false -Force
      if (-not $j.spellcheck) { $j | Add-Member -NotePropertyName spellcheck -NotePropertyValue ([pscustomobject]@{}) }
      $j.spellcheck | Add-Member -NotePropertyName dictionaries -NotePropertyValue @() -Force
      $json = $j | ConvertTo-Json -Depth 100 -Compress
      [IO.File]::WriteAllText($p, $json, (New-Object Text.UTF8Encoding $false))
      Write-Host "Da sua: $p"
    }
   ```
7. Kiểm tra kết quả. Sếp phải thấy các dòng “Da sua: ...”, mỗi file một dòng.
8. Mở lại Claude và gõ thử vài từ tiếng Việt.

## Lệnh này làm gì?
1. Đoạn lệnh tự sao lưu file gốc thành `Preferences.bak` trong cùng thư mục.
2. Nếu Claude không mở được hoặc lỗi, hãy thoát Claude, xóa file `Preferences` rồi đổi tên `Preferences.bak` thành `Preferences` để khôi phục. Hoặc chạy lệnh này:
   ```powershell
   Get-ChildItem "$env:LOCALAPPDATA\Claude-Data" -Recurse -Filter "Preferences.bak" -File | ForEach-Object { Copy-Item $_.FullName ($_.FullName -replace '\.bak$','') -Force }
   ```
4. Ứng dụng có thể ghi đè lại thiết lập này khi khởi động hoặc khi cập nhật. Khi đó, hãy làm lại các bước như ở trên.

## Ảnh chạy lệnh

<img width="1115" height="628" alt="Screenshot 2026-11-20 093657" src="https://github.com/user-attachments/assets/c45ad61e-0b6b-42b6-831f-0c40b2b14cf1" />
