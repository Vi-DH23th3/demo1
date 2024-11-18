import tkinter as tk
from tkinter import messagebox
from tkinter import ttk  # Sử dụng ttk cho combobox
from tkcalendar import Calendar
import sqlite3 #kết nối với sql
import pyodbc
from datetime import datetime
from reportlab.lib.pagesizes import letter
from reportlab.lib import colors
from reportlab.pdfgen import canvas

# tên và mật khẩu đăng nhập
Ten_Dangnhap = "tuong vi"
Matkhau_Dangnhap = "123456"

def thongtin_dangnhap():
    ten = entry_tenDN.get()
    mk = entry_mk.get()
    
    if ten == Ten_Dangnhap and mk == Matkhau_Dangnhap:
        open_form2()
    else:
        messagebox.showerror("Lỗi", "Tên đăng nhập hoặc mật khẩu không đúng")

def Cancel():
    root.destroy()
#hàm tính tiền và lưu hóa đơn cho form4 bán sách
def tinhtien(entry_gia, entry_sl, entry_tong):
    try:
        tiensach = float(entry_gia.get())  
        soluong = int(entry_sl.get())     
        sum = tiensach * soluong
        entry_tong.insert(0, str(sum))  
    except ValueError:
        messagebox.showerror("Lỗi", "Vui lòng nhập đúng số lượng và giá sách.")

def save_hoadon(tenkh, masach, soluong, gia, tongtien):
    if tenkh and masach and soluong and gia and tongtien:
        filename = f"HoaDon_invoice.pdf"
        c = canvas.Canvas(filename, pagesize=letter)

        c.setFont("Times New Roman-Bold", 16)
        c.drawString(100, 750, "HÓA ĐƠN")

        c.setFont("Times New Roman", 12)
        c.drawString(100, 730, f"Tên khách hàng: {tenkh}")
        c.drawString(100, 710, f"Tên sách: {masach}")
        c.drawString(100, 690, f"Số lượng: {soluong}")
        c.drawString(100, 670, f"Giá sách: {gia} VNĐ")
        c.drawString(100, 650, f"Tổng tiền: {tongtien} VNĐ")

        c.save()
        messagebox.showinfo("Thông báo", "Hóa đơn đã được lưu.")
    else:
        messagebox.showwarning("Cảnh báo", "Vui lòng nhập đầy đủ thông tin.")


# Cập nhật hàm lưu hóa đơn
def save_invoice(entry_tenkh, entry_sl, entry_tensach, entry_gia, entry_tong):
    # Lấy dữ liệu từ các ô nhập liệu
    tenkh = entry_tenkh.get()
    tensach = entry_tensach.get()
    soluong = int(entry_sl.get())
    giatien = float(entry_gia.get())
    tongtien = entry_tong.get()

    # Kiểm tra thông tin 
    if tenkh and tensach and soluong and giatien:
        save_hoadon(tenkh, tensach, soluong, giatien, tongtien)
    else:
        messagebox.showwarning("Cảnh báo", "Vui lòng điền đầy đủ thông tin.")
'''def hien_lich(label_selected_date,cal):
    #calendar = Calendar(selectmode='day', date_pattern='yyyy-mm-dd')
    selected_date = cal.get_date()  # Trả về ngày ở dạng 'yyyy-mm-dd'
    #year, month, day = map(int, selected_date.split('-'))

    label_selected_date.config(text=f"Ngày đã chọn: {selected_date}")  # Cập nhật label
    # In ra lịch của tháng đã chọn
   #print(f"Lịch tháng {month}, năm {year}:")
    #print(calendar.month(year, month))
'''
#form Trang chủ
def open_form2():
    root.withdraw()
    form2 = tk.Toplevel(root)
    form2.title("Quản lý sách")
    form2.geometry("400x200")

    
    # Tạo menu chính
    main_menu = tk.Menu(form2)

    # Nhập sách
    menu_nhapsach = tk.Menu(main_menu, tearoff=0)
    menu_nhapsach.add_command(label="Nhập Sách", command=open_form3)

    # Thống kê
    menu_ThongKe = tk.Menu(main_menu, tearoff=0)
    menu_ThongKe.add_command(label="Thống kê", command=open_form5)

    #Thoát
    menu_thoat = tk.Menu(main_menu, tearoff=0)
    menu_thoat.add_command(label="Thoát", command=form2.destroy)

    #Bán sách
    menu_Bansach = tk.Menu(main_menu, tearoff=0)
    menu_Bansach.add_cascade( label="Tính tiền sách", command=open_form4)

    # Thêm các menu con vào menu chính
    main_menu.add_cascade(label="Nhập sách", menu=menu_nhapsach)
    main_menu.add_cascade(label="Bán sách", menu=menu_Bansach)
    main_menu.add_cascade(label="Thống kê", menu=menu_ThongKe)
    main_menu.add_cascade(label="Thoát", menu=menu_thoat)

    form2.config(menu=main_menu)
# Kết nối và lấy dữ liệu từ cơ sở dữ liệu

# Chèn dữ liệu vào ListView

def them(tree, entry_sopn, entry_ms, entry_tens, entry_theloai, entry_tacgia, combobox_nxb, entry_sl, entry_gia): 
    try:
        ma_pn = entry_sopn.get()
        ma_sach = entry_ms.get()
        ten_sach = entry_tens.get()
        the_loai = entry_theloai.get()
        tacgia = entry_tacgia.get()
        nxb = combobox_nxb.get()
        so_luong = entry_sl.get()
        thanh_tien = entry_gia.get()

        # Kiểm tra nếu các trường đều có giá trị
        if not all([ma_pn, ma_sach, ten_sach, the_loai, tacgia, nxb, so_luong, thanh_tien]):
            messagebox.showwarning("Lỗi nhập liệu", "Vui lòng điền đầy đủ các trường.")
            return
        
        # Thêm dữ liệu vào Treeview
        tree.insert('', 'end', values=(ma_pn, ma_sach, ten_sach, the_loai, tacgia, nxb, so_luong, thanh_tien))

        # Xóa các trường nhập liệu
        entry_sopn.delete(0, 'end')
        entry_ms.delete(0, 'end')
        entry_tens.delete(0, 'end')
        entry_theloai.delete(0, 'end')
        entry_tacgia.delete(0, 'end')
        entry_sl.delete(0, 'end')
        entry_gia.delete(0, 'end')
    except Exception as e:
        messagebox.showerror("Lỗi", f"Đã xảy ra lỗi khi thêm dữ liệu: {e}")

def delete_data(tree):
    selected_item = tree.selection()
    if selected_item:
        tree.delete(selected_item)
    else:
        messagebox.showwarning("Chưa chọn", "Vui lòng chọn một dòng để xóa.")

#form nhập sách
def open_form3():
    form3 = tk.Toplevel(root)
    form3.title("Nhập sách")
    form3.geometry("820x624")

    label_header = tk.Label(form3, text="Phiếu nhập sách", font=("Times New Roman", 20),fg="blue")
    label_header.grid(row=0, column=0, columnspan=2, padx=10, pady=10)

    
# Tạo frame1
    frame = tk.Frame(form3)
    frame.grid(row=1, column=0, padx=20, pady=20)
# Tiêu đề của groupbox
    label_title1 = tk.Label(frame, text="Thông tin phiếu nhập", anchor="w")
    label_title1.grid(row=2, column=0, padx=5, pady=5, sticky="w")
# Tạo một groupbox của Thông tin phiếu nhập
    groupbox = tk.Frame(frame)
    groupbox.grid(row=3, rowspan=3)
    label_sophieu=tk.Label(groupbox,text="Số phiếu nhập")
    label_sophieu.grid(row=4,column=0)
    entry_sopn=tk.Entry(groupbox,width=20)
    entry_sopn.grid(row=4,column=1)
    
    label2=tk.Label(groupbox,text="Nhà xuất bản")
    label2.grid(row=4,column=2)
    combobox_nxb = ttk.Combobox(groupbox, values=["NXB Kim Đồng", "NXB Hội văn học", "NXB Đông Á"], width=20)
    combobox_nxb.grid(row=4, column=3, padx=5, pady=5)

    label_ngay=tk.Label(groupbox,text="Ngày nhập")
    label_ngay.grid(row=5,column=0)
    entry_selected_date = tk.Entry(groupbox,width=20)
    entry_selected_date.grid(row=5,column=1,pady=5)  
# Tạo Button để hiển thị ngày đã chọn
    btn_show = tk.Button(groupbox, text="Chọn ngày", command=lambda: hien_lich_form6(form3, entry_selected_date))
    btn_show.grid(row=6,column=0,pady=10)

    btn_save=tk.Button(groupbox,text="Lưu phiếu nhập")
    btn_save.grid(row=5,column=2)
    btn_them = tk.Button(groupbox, text="Them", command=lambda: them(tree, entry_sopn, entry_ms, entry_tens, entry_theloai, entry_tacgia, combobox_nxb, entry_sl, entry_gia))
    btn_them.grid(row=5, column=3)
    btn_delete = tk.Button(groupbox, text="Xóa", command=lambda: delete_data(tree))
    btn_delete.grid(row=5, column=4)
    # Tạo Frame thứ 2 
    frame = tk.Frame(form3)
    frame.grid(row=7, column=0, padx=20, pady=20)

# Tiêu đề của groupbox
    label_title2 = tk.Label(frame, text="Thông tin sách", anchor="w")
    label_title2.grid(row=7, column=0, padx=5, pady=5, sticky="w")

# Tạo một groupbox của Thông tin sách
    groupbox2 = tk.Frame(frame)
    groupbox2.grid(row=8, rowspan=5)
    label_ma=tk.Label(groupbox2,text="Mã sách")
    label_ma.grid(row=9,column=0)
    entry_ms=tk.Entry(groupbox2,width=30)
    entry_ms.grid(row=9,column=1)
    label_sl=tk.Label(groupbox2,text="Số lượng")
    label_sl.grid(row=10,column=0)
    entry_sl=tk.Entry(groupbox2,width=30)
    entry_sl.grid(row=10,column=1)
    label_gia=tk.Label(groupbox2,text="Thành tiền")
    label_gia.grid(row=11,column=0)
    entry_gia=tk.Entry(groupbox2,width=30)
    entry_gia.grid(row=11,column=1)

    label_ten=tk.Label(groupbox2,text="Tên sách")
    label_ten.grid(row=9,column=2,pady=5)
    entry_tens=tk.Entry(groupbox2,width=30)
    entry_tens.grid(row=9,column=3)
    label_tl=tk.Label(groupbox2,text="Thể loại")
    label_tl.grid(row=10,column=2)
    entry_theloai=tk.Entry(groupbox2,width=30)
    entry_theloai.grid(row=10,column=3)
    label_tg=tk.Label(groupbox2,text="Tác giả")
    label_tg.grid(row=11,column=2)
    entry_tacgia=tk.Entry(groupbox2,width=30)
    entry_tacgia.grid(row=11,column=3)
#Tạo frame3
    frame = tk.Frame(form3)
    frame.grid(row=12, column=0, padx=20, pady=20)

    label_title3 = tk.Label(frame, text="Chi tiết phiếu nhập", anchor="w")
    label_title3.grid(row=12, column=0, padx=5, pady=5, sticky="w")
    columns = ('Mã phiếu nhập', 'Mã sách', 'Tên sách','Thể loại','Tác giả','NXB','Số lượng','Thành tiền',)
    tree=ttk.Treeview(frame,columns=columns, show='headings')
    for col in columns:
        tree.heading(col, text=col)
        tree.column(col, width=140)
    tree.grid(row=14,padx=5, pady=5)
    #btn_add = tk.Button(frame, text="Thêm", command=lambda:them(tree,entry_sopn,entry_ms,entry_tens,entry_theloai,entry_tacgia,combobox_nxb,entry_sl,entry_gia))
    #btn_add.grid(row=15, column=0, padx=5, pady=5, sticky="w")

    
# Form4 Quản lý bán sách
def open_form4():
    form4 = tk.Toplevel(root)
    form4.title("Bán sách")
    form4.geometry("400x400")

    label_form4 = tk.Label(form4, text="Tính tiền sách", fg="blue", font=("Times New Roman", 20))
    label_form4.grid(row=0, column=0, rowspan=1, padx=10, pady=10)

    frame = tk.Frame(form4)
    frame.grid(row=1, column=0, padx=20, pady=20)

    label_title2 = tk.Label(frame, text="Hóa đơn", anchor="w")
    label_title2.grid(row=2, column=0, padx=5, pady=5, sticky="w")

   
    label_tenkh = tk.Label(frame, text="Tên khách hàng")
    label_tenkh.grid(row=3, column=0, padx=5, pady=5)
    entry_tenkh = tk.Entry(frame, width=30)
    entry_tenkh.grid(row=3, column=1, padx=5, pady=5)

    label_tens = tk.Label(frame, text="Tên sách")
    label_tens.grid(row=4, column=0, padx=5, pady=5)
    entry_tensach = tk.Entry(frame, width=30)
    entry_tensach.grid(row=4, column=1, padx=5, pady=5)

    label_ma = tk.Label(frame, text="Mã sách")
    label_ma.grid(row=5, column=0, padx=5, pady=5)
    entry_ms = tk.Entry(frame, width=30)
    entry_ms.grid(row=5, column=1, padx=5, pady=5)

    label_sl = tk.Label(frame, text="Số lượng")
    label_sl.grid(row=6, column=0, padx=5, pady=5)
    entry_sl = tk.Entry(frame, width=30)  
    entry_sl.grid(row=6, column=1, padx=5, pady=5)

    label_giasach = tk.Label(frame, text="Giá sách:")
    label_giasach.grid(row=7, column=0, padx=5, pady=5)
    entry_gia = tk.Entry(frame, width=30)  
    entry_gia.grid(row=7, column=1, padx=5, pady=5)

    label_tong = tk.Label(frame, text="Tổng")
    label_tong.grid(row=8, column=0, padx=5, pady=5)
    entry_tong = tk.Entry(frame, width=30)  
    entry_tong.grid(row=8, column=1, padx=5, pady=5)

    btn_tinhtien = tk.Button(frame, text="Tính tiền", command=lambda: tinhtien(entry_gia, entry_sl, entry_tong))
    btn_tinhtien.grid(row=9, column=0, pady=5,padx=5)

   
    btn_save = tk.Button(frame, text="Lưu hóa đơn", command=lambda: save_invoice(entry_tenkh, entry_sl, entry_tensach, entry_gia, entry_tong))
    btn_save.grid(row=9, column=1, pady=5,padx=5)

    btn_thoat = tk.Button(frame, text="Thoát", command=form4.destroy)
    btn_thoat.grid(row=9, column=2, padx=5, pady=5)

# Form5 Thống kê
def open_form5():
    form5 = tk.Toplevel(root)
    form5.title("Thống kê")
    form5.geometry("300x300")
    label_title=tk.Label(form5,text="Thông kê",font=("Times New Roman",16),fg="blue")
    label_title.grid(row=0,rowspan=1)

    frame4=tk.Frame(form5)
    frame4.grid(row=1, column=0, padx=20, pady=20)

    label_ngay=tk.Label(frame4,text="Chọn ngày")
    label_ngay.grid(row=2,column=0,padx=5,pady=5)
    entry_chonngay=tk.Entry(frame4,width=20)
    entry_chonngay.grid(row=2,column=1,padx=5,pady=5)

    btn_tim=tk.Button(frame4,text="Tìm")
    btn_tim.grid(row=1,column=2,padx=5,pady=5)

    btn_thoat=tk.Button(frame4,text="Thoát",command=form5.destroy)
    btn_thoat.grid(row=2,column=2,padx=5,pady=5)
def hien_lich(entry_selected_date, cal):
    # Lấy ngày đã chọn từ Calendar (dạng 'yyyy-mm-dd')
    selected_date = cal.get_date()
    
    # Cập nhật label với ngày đã chọn
    #entry_selected_date.config(text=f"Ngày đã chọn:{selected_date}")  
    entry_selected_date.config(selected_date)  
def hien_lich_form6(form3, entry_selected_date):
    form6= tk.Toplevel(form3)
    form6.title("Chọn ngày")
    form6.geometry("300x300")
    
    cal=Calendar(form6,selectmode='day',date_pattern='dd-mm-yyyy')
    cal.grid(pady=20)

    def get_selected_date():
        selected_date = cal.get_date()  # Lấy ngày chọn từ Calendar
        # Cập nhật giá trị vào entry_selected_date
        entry_selected_date.delete(0, tk.END)  # Xóa giá trị cũ
        entry_selected_date.insert(0, selected_date)  # Chèn ngày mới vào Entry
        form6.destroy()  

    btn_select_date = tk.Button(form6, text="Chọn ngày", command=get_selected_date)
    btn_select_date.grid(row=1,column=0)

# Create the main window
root = tk.Tk()
root.title("Login")
root.minsize(300, 200)

# Create the form (login form)
label_header = tk.Label(root, text="Đăng Nhập", font=("Times New Roman", 20),fg="blue")
label_header.grid(row=0, column=0, columnspan=2)

# Tên đăng nhập label and entry
label_tenDN = tk.Label(root, text="Tên đăng nhập")
label_tenDN.grid(row=1, column=0, padx=10, pady=5, sticky="e")
entry_tenDN = tk.Entry(root, width=30)
entry_tenDN.grid(row=1, column=1, padx=10, pady=5)

# Mật khẩu label and entry
label_MK = tk.Label(root, text="Mật Khẩu")
label_MK.grid(row=2, column=0, padx=10, pady=5, sticky="e")
entry_mk = tk.Entry(root, width=30, show="*")
entry_mk.grid(row=2, column=1, padx=10, pady=5)

# Button frame
frameButton = tk.Frame(root)
frameButton.grid(row=3, columnspan=2, pady=10)

btn_ok = tk.Button(frameButton, text="OK", command=thongtin_dangnhap)
btn_ok.pack(side="left", padx=10)

btn_cancel = tk.Button(frameButton, text="Cancel", command=Cancel)
btn_cancel.pack(side="left", padx=10)

# Start the Tkinter event loop
root.mainloop()
