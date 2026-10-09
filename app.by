import streamlit as st
import pandas as pd
import os

# Cấu hình giao diện Web
st.set_page_config(page_title="Hệ thống Bầu cử Trực tuyến", layout="wide", page_icon="🗳️")

st.title("🗳️ HỆ THỐNG BẦU CỬ TRỰC TUYẾN DÀNH CHO MỌI NGƯỜI")
st.markdown("---")

# Định nghĩa file lưu trữ dữ liệu chung trên máy chủ Streamlit
DATA_FILE = "ket_qua_bau_cu.csv"

# Danh sách ứng viên cố định (Bạn có thể sửa tên ứng viên trực tiếp ở đây trước khi đưa lên mạng)
DANH_SACH_UV = [
    {"id": 1, "ho_ten": "Nguyễn Văn A"},
    {"id": 2, "ho_ten": "Trần Thị B"},
    {"id": 3, "ho_ten": "Lê Văn C"},
    {"id": 4, "ho_ten": "Phạm Minh D"},
    {"id": 5, "ho_ten": "Hoàng Thị E"}
]
SO_LUONG_BAU_TOI_DA = 3 # Số người được bầu tối đa trên 1 phiếu

# Hàm đọc dữ liệu từ file chung
def doc_du_lieu():
    if os.path.exists(DATA_FILE):
        return pd.read_csv(DATA_FILE)
    else:
        # Nếu chưa có file, tạo file mới với các cột ban đầu bằng 0
        df = pd.DataFrame([{f"uv_{uv['id']}": 0 for uv in DANH_SACH_UV} | {"hop_le": 0, "khong_hop_le": 0}])
        df.to_csv(DATA_FILE, index=False)
        return df

# Hàm ghi dữ liệu mới (Cập nhật phiếu bầu)
def ghi_du_lieu(bi_gach_list, la_hop_le):
    df = doc_du_lieu()
    if la_hop_le:
        df["hop_le"] += 1
        for uv in DANH_SACH_UV:
            if uv["id"] not in bi_gach_list:
                df[f"uv_{uv['id']}"] += 1
    else:
        df["khong_hop_le"] += 1
    df.to_csv(DATA_FILE, index=False)

# Đọc dữ liệu hiện tại từ hệ thống
df_hien_tai = doc_du_lieu()

# Tách giao diện làm 2 cột: Cột bầu cử và Cột kết quả
col1, col2 = st.columns([1, 1.2])

with col1:
    st.subheader("📥 Khu vực bỏ phiếu")
    st.info(f"💡 Hướng dẫn: Tích chọn vào những người bạn muốn **GẠCH TÊN** (Không bầu). Bạn được bầu tối đa {SO_LUONG_BAU_TOI_DA} người.")
    
    # Sử dụng Form để tránh việc tải lại trang khi đang bấm chọn
    with st.form(key="form_bau_cu", clear_on_submit=True):
        bi_gach_list = []
        for uv in DANH_SACH_UV:
            checked = st.checkbox(f"[{uv['id']}] {uv['ho_ten']}", key=f"form_gach_{uv['id']}")
            if checked:
                bi_gach_list.append(uv['id'])
                
        submit = st.form_submit_button("💾 Gửi phiếu bầu của bạn", type="primary")
        
        if submit:
            so_luong_gach = len(bi_gach_list)
            so_luong_bau = len(DANH_SACH_UV) - so_luong_gach
            
            # Kiểm tra tính hợp lệ
            la_hop_le = True
            if so_luong_bau > SO_LUONG_BAU_TOI_DA or so_luong_bau <= 0:
                la_hop_le = False
                st.error(f"Phiếu không hợp lệ! Bạn đang bầu cho {so_luong_bau} người (Quy định là từ 1 đến {SO_LUONG_BAU_TOI_DA} người). Phiếu này vẫn được ghi nhận là phiếu không hợp lệ.")
            else:
                st.success("Cảm ơn bạn! Phiếu bầu hợp lệ đã được gửi thành công.")
                
            # Lưu vào hòm phiếu chung
            ghi_du_lieu(bi_gach_list, la_hop_le)
            st.rerun()

with col2:
    st.subheader("📊 Kết quả tổng hợp (Cập nhật liên tục)")
    
    phieu_hl = int(df_hien_tai["hop_le"].iloc[0])
    phieu_khl = int(df_hien_tai["khong_hop_le"].iloc[0])
    tong_phieu = phieu_hl + phieu_khl
    
    m1, m2, m3 = st.columns(3)
    m1.metric("Tổng số người đã bầu", tong_phieu)
    m2.metric("Số phiếu HỢP LỆ", phieu_hl)
    m3.metric("Số phiếu KHÔNG HỢP LỆ", phieu_khl)
    
    # Hiển thị bảng xếp hạng ứng viên
    st.markdown("#### Bảng số phiếu của các ứng viên")
    data_bang = []
    for uv in DANH_SACH_UV:
        so_phieu = int(df_hien_tai[f"uv_{uv['id']}"].iloc[0])
        ty_le = (so_phieu / phieu_hl * 100) if phieu_hl > 0 else 0.0
        data_bang.append({
            "STT": uv['id'],
            "Họ và Tên": uv['ho_ten'],
            "Số phiếu bầu đạt được": so_phieu,
            "Tỷ lệ % (trên phiếu HL)": f"{ty_le:.2f} %"
        })
        
    df_ket_qua = pd.DataFrame(data_bang).sort_values(by="Số phiếu bầu đạt được", ascending=False)
    st.dataframe(df_ket_qua, use_container_width=True, hide_index=True)
