import tkinter as tk
from tkinter import messagebox, simpledialog
from tkinter import ttk  # Sử dụng ttk cho combobox
from tkcalendar import Calendar
import sqlite3 #kết nối với sql
import pyodbc
import pandas as pd # type: ignore
from datetime import datetime
from reportlab.lib.pagesizes import letter
from reportlab.lib import colors
from reportlab.pdfgen import canvas
##THIẾT KẾ FORM ĐĂNG NHẬP
# tên và mật khẩu đăng nhập
Ten_Dangnhap = "admin"
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

##THIẾT KẾ CÁC PHƯƠNG THỨC CHO FORM3: NHẬP SÁCH
def them(tree, entry_ms, entry_tens, entry_theloai, entry_tacgia, combobox_nxb, entry_sl, entry_gia): 
    try:
       
        ma_sach = entry_ms.get()
        ten_sach = entry_tens.get()
        the_loai = entry_theloai.get()
        tacgia = entry_tacgia.get()
        nxb = combobox_nxb.get()
        so_luong = entry_sl.get()
        thanh_tien = entry_gia.get()

        # Kiểm tra nếu các trường đều có giá trị
        if not all([ ma_sach, ten_sach, the_loai, tacgia, nxb, so_luong, thanh_tien]):
            messagebox.showwarning("Lỗi nhập liệu", "Vui lòng điền đầy đủ các dữ liệu.")
            return
        
        # Thêm dữ liệu vào Treeview
        tree.insert('', 'end', values=( ma_sach, ten_sach, the_loai, tacgia, nxb, so_luong, thanh_tien))

        # Xóa các trường nhập liệu
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
     # Kiểm tra xem người dùng đã chọn dòng nào chưa
    if not selected_item:
        messagebox.showwarning("Chưa chọn", "Hãy chọn một dòng để xóa.")
        return    
    # Hiển thị hộp thoại xác nhận xóa
    confirm = messagebox.askyesno("Xác nhận xóa", "Bạn có chắc chắn muốn xóa dòng này?")   
    if confirm:  # Nếu người dùng chọn "Yes"
        tree.delete(selected_item)
        messagebox.showinfo("Xóa thành công", "Dữ liệu đã được xóa.")
def sua(tree, entry_ms, entry_tens, entry_theloai, entry_tacgia, combobox_nxb,entry_selected_date, entry_sl, entry_gia):
    selected_item = tree.selection()
    if selected_item:
        # Lấy giá trị của dòng đã chọn
        item_values = tree.item(selected_item)["values"]

        # Điền giá trị vào các widget

        entry_ms.delete(0, tk.END)
        entry_ms.insert(0, item_values[0])  # Mã sách

        entry_tens.delete(0, tk.END)
        entry_tens.insert(0, item_values[1])  # Tên sách

        entry_theloai.delete(0, tk.END)
        entry_theloai.insert(0, item_values[2])  # Thể loại

        entry_tacgia.delete(0, tk.END)
        entry_tacgia.insert(0, item_values[3])  # Tác giả

        combobox_nxb.set(item_values[4])  # NXB (nếu sử dụng combobox)

        entry_sl.delete(0, tk.END)
        entry_sl.insert(0, item_values[5])  # Số lượng

        entry_gia.delete(0, tk.END)
        entry_gia.insert(0, item_values[6])  # Thành tiền
def save_phieunhap(tree, entry_sopn, entry_ms, entry_tens, entry_theloai, entry_tacgia, combobox_nxb,entry_selected_date, entry_sl, entry_gia):
    # Lấy giá trị từ các trường nhập liệu (thông tin phiếu nhập)
    sopn = entry_sopn.get()  # Số phiếu nhập
    #nxb = combobox_nxb.get()  # Nhà xuất bản
    ngay_nhap = entry_selected_date.get()  # Ngày nhập

    # Kiểm tra nếu các trường quan trọng trống
    if not sopn or not ngay_nhap :
        messagebox.showwarning("Cảnh báo", "Vui lòng điền đầy đủ thông tin phiếu nhập!")
        return

    # Tạo tên file PDF dựa trên số phiếu nhập
    file_name = f"phieu_nhap_sach_{sopn}.pdf"

    # Tạo đối tượng Canvas từ thư viện reportlab
    c = canvas.Canvas(file_name, pagesize=letter)
    #c.setFont("FreeSerif", 12) 
    # Thiết lập font và kích thước chữ
   
    c.setFont("Times New Roman-Bold", 16)
    # Tiêu đề phiếu nhập sách
    c.drawString(200, 750, "PHIEU NHAP SACH")
    c.setFont("Times New Roman", 12)
    # Thông tin phiếu nhập
    c.drawString(50, 700, f"Số phiếu nhập: {sopn}")
    c.drawString(50, 680, f"Ngày nhập: {ngay_nhap}")
    #c.drawString(50, 660, f"Nhà xuất bản: {nxb}")

    # Dữ liệu từ Treeview - Tiêu đề cột
    columns = ('Mã sách', 'Tên sách', 'Thể loại', 'Tác giả', 'NXB', 'Số lượng', 'Thành tiền')
    
    # Vẽ tiêu đề cột
    y_position = 620  # Vị trí y bắt đầu cho tiêu đề
    for col in columns:
        c.drawString(50, y_position, col)
        y_position -= 20  # Khoảng cách giữa các dòng tiêu đề

    # Vẽ dữ liệu từ Treeview
    for child in tree.get_children():
        row = tree.item(child)["values"]
        y_position -= 20  # Dịch xuống cho mỗi dòng dữ liệu
        c.drawString(50, y_position, str(row[0]))  # Mã sách
        c.drawString(150, y_position, str(row[1]))  # Tên sách
        c.drawString(250, y_position, str(row[2]))  # Thể loại
        c.drawString(350, y_position, str(row[3]))  # Tác giả
        c.drawString(450, y_position, str(row[4]))  # Nhà xuất bản
        c.drawString(550, y_position, str(row[5]))  # Số lượng
        c.drawString(650, y_position, str(row[6]))  # Thành tiền
    # Lưu file PDF
    c.save()

    # Thông báo khi lưu thành công
    messagebox.showinfo("Thông báo", f"Phiếu nhập đã được lưu thành công vào file {file_name}")
    
    # Xóa các trường sau khi đã lưu
    entry_sopn.delete(0, tk.END)
    combobox_nxb.set('')
    entry_selected_date.delete(0, tk.END)
    entry_ms.delete(0, tk.END)
    entry_tens.delete(0, tk.END)
    entry_theloai.delete(0, tk.END)
    entry_tacgia.delete(0, tk.END)
    entry_sl.delete(0, tk.END)
    entry_gia.delete(0, tk.END)
#Phương thức cho phép người dùng chọn ngày
def hien_lich(entry_selected_date, cal):
    # Lấy ngày đã chọn từ Calendar (dạng 'yyyy-mm-dd')
    selected_date = cal.get_date()
    
    # Cập nhật label với ngày đã chọn
    #entry_selected_date.config(text=f"Ngày đã chọn:{selected_date}")  
    entry_selected_date.config(selected_date)  
def hien_lich_form6(form3, entry_selected_date):
    form6= tk.Toplevel(form3)
    form6.title("Chọn ngày")
    form6.geometry("400x300")
    
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
#form nhập sách
def open_form3():
    form3 = tk.Toplevel(root)
    form3.title("Nhập sách")
    form3.geometry("1100x750")
   
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

    btn_save=tk.Button(groupbox,text="Lưu phiếu nhập",command=lambda:save_phieunhap(tree, entry_sopn, entry_ms, entry_tens, entry_theloai, entry_tacgia, combobox_nxb,entry_selected_date, entry_sl, entry_gia))
    btn_save.grid(row=5,column=2)
    btn_them = tk.Button(groupbox, text="Them", command=lambda: them(tree, entry_ms, entry_tens, entry_theloai, entry_tacgia, combobox_nxb, entry_sl, entry_gia))
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
    columns = ('Mã sách', 'Tên sách','Thể loại','Tác giả','NXB','Số lượng','Thành tiền',)
    #tree=ttk.Treeview(frame,columns=columns, show='headings')
    tree=ttk.Treeview(frame,columns=columns, show='headings')
    for col in columns:
        tree.heading(col, text=col)
        tree.column(col, width=140)
    tree.grid(row=14,padx=5, pady=5)

    # Thêm dữ liệu mẫu vào Treeview
    tree.insert("", "end", values=( "S001", "Python Basics", "Lập trình", "Nguyễn Văn A", "NXB ABC", 10, 200000))
    tree.insert("", "end", values=( "S002", "Thám tử lừng danh conan", "Trinh thám", "Gosho Aoyama", "NXB Kim Đồng", 5, 450000))
    
##THIẾT KẾ CÁC PHƯƠNG THỨC CHO FORM4
#hàm tính tiền và lưu hóa đơn cho form4 bán sách
def tinhtien(entry_gia, entry_sl, entry_tong):
    try:
        tiensach = float(entry_gia.get())  
        soluong = int(entry_sl.get())     
        sum = tiensach * soluong
        entry_tong.delete(0,tk.END)
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
def save_data_to_sql(entry_madonhang, entry_tenkh, entry_ngayban, entry_ms, entry_sl, entry_gia, entry_tong):
    server = "DESKTOP-EL1NO5U\\SQLSERVER1"
    database = "quanlysach"
    username = "vi"
    password = "123456"
    connection_string = f'DRIVER={{SQL Server}};SERVER={server};DATABASE={database};UID={username};PWD={password};CHARSET=UTF8'
    
    try:
        # Kết nối đến cơ sở dữ liệu
        conn = pyodbc.connect(connection_string)
        cursor = conn.cursor()

        # Lấy dữ liệu từ các ô nhập liệu
        madonhang = entry_madonhang.get()  # Mã đơn hàng
        tenkh_str = entry_tenkh.get()  # Tên khách hàng
        #tenkh=tenkh_utf8.decode('utf-8')        
        ngayban_str = entry_ngayban.get()  # Ngày bán (dưới dạng chuỗi)
        masach = entry_ms.get()            # Mã sách
        soluong = int(entry_sl.get())      # Số lượng bán
        giaban = float(entry_gia.get())    # Giá bán
        tongtien = float(entry_tong.get()) # Tổng tiền

         # Chuyển đổi ngày bán từ chuỗi sang kiểu ngày (DATE) của SQL
        try:
            # Giả sử người dùng nhập ngày theo định dạng "dd-mm-yyyy"
            ngayban = datetime.strptime(ngayban_str, "%d-%m-%Y").date()
        except ValueError:
            print("Lỗi: Định dạng ngày không hợp lệ. Vui lòng nhập ngày theo định dạng dd-mm-yyyy.")
            return  # Nếu ngày không hợp lệ, thoát hàm

        # Kiểm tra xem masach có tồn tại trong bảng books không
        cursor.execute("SELECT COUNT(*) FROM books WHERE masach = ?", (masach,))
        if cursor.fetchone()[0] == 0:
            print(f"Lỗi: Mã sách {masach} không tồn tại trong bảng books.")
            return 
         # Kiểm tra xem mã đơn hàng có tồn tại trong bảng qlbanhang không
        cursor.execute("SELECT COUNT(*) FROM qlbanhang WHERE madonhang = ?", (madonhang,))
        if cursor.fetchone()[0] > 0:
            print(f"Lỗi: Mã đơn hàng {madonhang} đã tồn tại.")
            return  # Nếu mã đơn hàng đã tồn tại, thoát hàm
         # Thêm dữ liệu vào bảng qlbanhang
        cursor.execute("""
            INSERT INTO qlbanhang (madonhang, tenkh, ngayban, tongtien) 
            VALUES (?, ?, ?, ?)
        """, (madonhang, tenkh_str, ngayban, tongtien))

        # Thêm dữ liệu vào bảng chitiet
        cursor.execute("""
            INSERT INTO chitiet (madonhang, masach, soluong, giaban) 
            VALUES (?, ?, ?, ?)
        """, (madonhang, masach, soluong, giaban))

        # Commit các thay đổi vào cơ sở dữ liệu
        conn.commit()


    except pyodbc.Error as e:
        # Nếu có lỗi trong quá trình kết nối hoặc thực thi SQL
        print("Lỗi khi kết nối hoặc thực thi SQL:", e)

    finally:
        # Đảm bảo rằng kết nối được đóng lại sau khi thao tác xong
        if conn:
            conn.close()

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
# Form4 Quản lý bán sách
def open_form4():
    form4 = tk.Toplevel(root)
    form4.title("Bán sách")
    form4.geometry("500x500")  # Điều chỉnh kích thước cửa sổ cho hợp lý hơn
    
    # Tiêu đề
    label_form4 = tk.Label(form4, text="Tính tiền sách", fg="blue", font=("Times New Roman", 20))
    label_form4.grid(row=0, columnspan=4, pady=10)  # Tiêu đề span qua tất cả các cột
    
    # Mã đơn hàng
    label_madonhang = tk.Label(form4, text="Mã đơn hàng")
    label_madonhang.grid(row=1, column=0, padx=5, pady=5, sticky="w")
    entry_madonhang = tk.Entry(form4, width=20)
    entry_madonhang.grid(row=1, column=1, padx=5, pady=5)

    # Ngày bán
    label_ngayban = tk.Label(form4, text="Ngày bán")
    label_ngayban.grid(row=1, column=2, padx=5, pady=5, sticky="w")
    entry_ngayban = tk.Entry(form4, width=20)
    entry_ngayban.grid(row=1, column=3, padx=5, pady=5)

    # Frame cho các thông tin hóa đơn
    frame = tk.Frame(form4)
    frame.grid(row=2, column=0, columnspan=4, padx=20, pady=20)

    # Tiêu đề bảng hóa đơn
    label_title2 = tk.Label(frame, text="Hóa đơn", anchor="w")
    label_title2.grid(row=0, column=0, padx=5, pady=5, sticky="w")

    # Tên khách hàng
    label_tenkh = tk.Label(frame, text="Tên khách hàng")
    label_tenkh.grid(row=1, column=0, padx=5, pady=5, sticky="w")
    entry_tenkh = tk.Entry(frame, width=30)
    entry_tenkh.grid(row=1, column=1, padx=5, pady=5)

    # Tên sách
    label_tens = tk.Label(frame, text="Tên sách")
    label_tens.grid(row=2, column=0, padx=5, pady=5, sticky="w")
    entry_tensach = tk.Entry(frame, width=30)
    entry_tensach.grid(row=2, column=1, padx=5, pady=5)

    # Mã sách
    label_ma = tk.Label(frame, text="Mã sách")
    label_ma.grid(row=3, column=0, padx=5, pady=5, sticky="w")
    entry_ms = tk.Entry(frame, width=30)
    entry_ms.grid(row=3, column=1, padx=5, pady=5)

    # Số lượng
    label_sl = tk.Label(frame, text="Số lượng")
    label_sl.grid(row=4, column=0, padx=5, pady=5, sticky="w")
    entry_sl = tk.Entry(frame, width=30)
    entry_sl.grid(row=4, column=1, padx=5, pady=5)

    # Giá sách
    label_giasach = tk.Label(frame, text="Giá sách:")
    label_giasach.grid(row=5, column=0, padx=5, pady=5, sticky="w")
    entry_gia = tk.Entry(frame, width=30)
    entry_gia.grid(row=5, column=1, padx=5, pady=5)

    # Tổng tiền
    label_tong = tk.Label(frame, text="Tổng:")
    label_tong.grid(row=6, column=0, padx=5, pady=5, sticky="w")
    entry_tong = tk.Entry(frame, width=30)
    entry_tong.grid(row=6, column=1, padx=5, pady=5)

    # Các nút chức năng
    btn_tinhtien = tk.Button(frame, text="Tính tiền", command=lambda: tinhtien(entry_gia, entry_sl, entry_tong))
    btn_tinhtien.grid(row=7, column=0, padx=5, pady=5,sticky="w")

    btn_savefile = tk.Button(frame, text="Lưu file", command=lambda: save_invoice(entry_madonhang, entry_tenkh, entry_ngayban, entry_tong))
    btn_savefile.grid(row=7, column=1, padx=5, pady=5,sticky="w")

    btn_save = tk.Button(frame, text="Lưu vào csdl", command=lambda:save_data_to_sql(entry_madonhang, entry_tenkh, entry_ngayban, entry_ms, entry_sl, entry_gia, entry_tong))
    btn_save.grid(row=7, column=2, padx=5, pady=5,sticky="w")
    btn_thoat = tk.Button(frame, text="Thoát", command=form4.destroy)
    btn_thoat.grid(row=7, column=3, padx=5, pady=5,sticky="w")
##THIẾT KẾ CÁC PHƯƠNG THỨC CHO FORM5: THỐNG KÊ
def load_data(tree):
    server = "DESKTOP-EL1NO5U\\SQLSERVER1"
    database = "quanlysach"
    username = "vi"
    password = "123456"
    connection_string = f'DRIVER={{SQL Server}};SERVER={server};DATABASE={database};UID={username};PWD={password};CHARSET=UTF8'
    # Xóa dữ liệu cũ trên TreeView
    for item in tree.get_children():
        tree.delete(item)

    try:
        conn = pyodbc.connect(connection_string)
        cursor = conn.cursor()
        cursor.execute("""
        SELECT qlbanhang.madonhang, 
           qlbanhang.tenkh, 
           qlbanhang.ngayban, 
           qlbanhang.tongtien,
           chitiet.masach, 
           chitiet.soluong
    FROM qlbanhang
    INNER JOIN chitiet ON qlbanhang.madonhang = chitiet.madonhang
""")
        rows = cursor.fetchall()

        # Thêm từng cột và giá trị tương ứng
        for row in rows:
            tree.insert("", "end", values=(
                row[0],  #mã đơn hàng
                row[1],  # tên khách hàng
                row[2],  # Ngày bán
                row[3],  # tổng tiền
                row[4],  # Mã sách
                row[5],  # Số lượng
            ))

        conn.close()
    except Exception as e:
        messagebox.showerror("Lỗi", f"Không thể tải dữ liệu: {e}")
# Hàm mở cửa sổ thống kê
def open_form5():
    form5 = tk.Toplevel(root)
    form5.title("Thống kê")
    form5.geometry("800x350")  # Điều chỉnh kích thước cửa sổ theo ý bạn
    
    # Thêm tiêu đề cho form
    label_title = tk.Label(form5, text="Thống kê", font=("Times New Roman", 16), fg="blue")
    label_title.grid(row=0, rowspan=1)

    tree = ttk.Treeview(form5, columns=("Mã đơn hàng","Tên khách hàng","Ngày bán","Tổng tiền","Mã sách","Số lượng"), show="headings")
    tree.grid(row=1,column=0)

    columns = ["Mã đơn hàng","Tên khách hàng","Ngày bán","Tổng tiền","Mã sách","Số lượng"]
    for col, title in zip(tree["columns"], columns):
        tree.heading(col, text=col)
        tree.column(col, width=140)
    
    load_data(tree)
    # Thêm nút thoát
    btn_thoat = tk.Button(form5, text="Thoát", command=form5.destroy)
    btn_thoat.grid(row=3, column=0)
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

root = tk.Tk()
root.title("Login")
root.minsize(300, 200)

label_header = tk.Label(root, text="Đăng Nhập", font=("Times New Roman", 20),fg="blue")
label_header.grid(row=0, column=0, columnspan=2)

label_tenDN = tk.Label(root, text="Tên đăng nhập")
label_tenDN.grid(row=1, column=0, padx=10, pady=5, sticky="e")
entry_tenDN = tk.Entry(root, width=30)
entry_tenDN.grid(row=1, column=1, padx=10, pady=5)

label_MK = tk.Label(root, text="Mật Khẩu")
label_MK.grid(row=2, column=0, padx=10, pady=5, sticky="e")
entry_mk = tk.Entry(root, width=30, show="*")
entry_mk.grid(row=2, column=1, padx=10, pady=5)

frameButton = tk.Frame(root)
frameButton.grid(row=3, columnspan=2, pady=10)

btn_ok = tk.Button(frameButton, text="OK", command=thongtin_dangnhap)
btn_ok.pack(side="left", padx=10)

btn_cancel = tk.Button(frameButton, text="Cancel", command=Cancel)
btn_cancel.pack(side="left", padx=10)

root.mainloop()
