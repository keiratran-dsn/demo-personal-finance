# Demo Personal Finance — Trung tâm tài chính

Prototype HTML 2 màn **Mặc định** và **Scroll** (Figma: Tài chính thông minh, node 266-22458), dựng trong khung iPhone, có animation.

- `public/index.html` — prototype (một file, không cần build)
- `tai-assets.sh` — tải icon/ảnh từ Figma vào `public/assets` (Vercel chạy khi build)

Chạy local: `bash tai-assets.sh` rồi mở `public/index.html`.

## Tham số URL
- `?tang=1|2a|2b|3` — tầng khách hàng (mặc định 3)
- `?tang=1&tk=xacthuc|quay|xetduyet|nangcap|kichhoat` — Tầng 1: thực trạng tài khoản quyết định popup Onboarding VTM khi bấm vào dịch vụ (Figma Onboarding `OOxHqzRWucPK5yCKCEPjMr` node `12548:266350`)
