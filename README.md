```mermaid
graph TD
    A[Bắt đầu - Mở trang Users Module] --> B{Kiểm tra Authentication};
    B -- Logged In --> C{Xác định trang hiện tại};
    B -- Not Logged In --> R[Chuyển hướng đến Login] --> Z[Kết thúc];

    C --> D{Loại trang?};
    D -- list --> E[initUsersList];
    D -- view --> F[initUserView];
    D -- edit --> G[initUserEdit];
    D -- create --> H[initUserCreate];
    D -- Khác --> Z;

    subgraph "List Page Workflow"
        E --> E1[Hiển thị Loading];
        E1 --> E2{Kiểm tra cờ 'needsRefresh'};
        E2 --> E3[Fetch Users Data];
        E3 --> E4[Call API GET /api/users];
        E4 --> E5{API Thành công?};
        E5 -- Yes --> E6[Xử lý dữ liệu, Cập nhật Cache];
        E5 -- No --> E7[Xử lý lỗi, Dùng Cache];
        E7 --> E6;
        E6 --> E8{Dùng fetch hoặc Cache?};
        E8 -- Fetch --> E9[Khởi tạo DataTable với AJAX];
        E8 -- Cache --> E10[Khởi tạo DataTable với Cache];
        E9 --> E11[Lưu trữ users];
        E10 --> E11;
        E11 --> E12[Thiết lập Event Listeners];
        E12 --> E13[Ẩn Loading];
        E13 --> E14[Hiển thị Bảng Users];
        E14 --> E15{Người dùng tương tác?};
        E15 -- Filter --> E16[Áp dụng Filter] --> E17[Vẽ lại Table] --> E14;
        E15 -- Delete --> E18{Xác nhận xóa?};
        E18 -- Yes --> E19[Gọi deleteUser];
        E19 --> E20[Call API DELETE /api/users];
        E20 --> E21{API Thành công?};
        E21 --> E22[Xóa user, Cập nhật Cache];
        E22 --> E23[Reload Table];
        E23 --> E24[Hiển thị thông báo thành công] --> E14;
        E18 -- No --> E14;
        E15 -- View/Edit --> E25[Chuyển đến View/Edit] --> Z;
        E15 -- Create --> E26[Chuyển đến Create] --> Z;
    end

    subgraph "View Page Workflow"
        F --> F1[Lấy userId từ URL];
        F1 --> F2[Hiển thị Loading];
        F2 --> F3[Fetch User Details];
        F3 --> F4[Call API GET /api/users/:id];
        F4 --> F5{API Thành công?};
        F5 -- Yes --> F6[Lưu dữ liệu user];
        F5 -- No --> F7[Xử lý lỗi, Dùng Cache];
        F7 --> F6;
        F6 --> F8[Fetch User Groups];
        F8 -- Cần fetch --> F9[Call fetchGroupUsers];
        F9 --> F10{API Thành công?};
        F10 -- Yes --> F11[Lưu Groups Data];
        F10 -- No --> F12[Xử lý lỗi];
        F12 --> F11;
        F8 -- Có data --> F11;
        F11 --> F13[Lọc group của user];
        F13 --> F14[Render User Details];
        F14 --> F15[Render Teams của User];
        F15 --> F16[Thiết lập Event Listeners];
        F16 --> F17[Ẩn Loading];
        F17 --> F18[Hiển thị User và Teams];
        F18 --> F19{Người dùng tương tác?};
        F19 -- Edit --> E25;
        F19 -- Delete --> E18V{Xác nhận xóa?};
        E18V -- Yes --> E19V[Gọi deleteUser];
        E19V --> E20V[Call API DELETE];
        E20V --> E21V{API Thành công?};
        E21V --> E22V[Xóa user, Cập nhật Cache];
        E22V --> E23V[Hiển thị thông báo thành công];
        E23V --> E24V[Chuyển hướng đến List] --> Z;
        E18V -- No --> F18;
        F19 -- Remove Team --> F20{Xác nhận xóa?};
        F20 -- Yes --> F21[Call removeUserFromTeam];
        F21 --> F22[Call API DELETE];
        F22 --> F23{API Thành công?};
        F23 --> F24[Xóa liên kết, Cập nhật Cache];
        F24 --> F25[Reload Teams Table];
        F25 --> F26[Hiển thị thông báo thành công] --> F18;
        F20 -- No --> F18;
    end
