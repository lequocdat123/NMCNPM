using CoffeShopManagerment.DAO;
using CoffeShopManagerment.DTO;
using CoffeShopManagerment.Utils;
using System;
using System.Collections.Generic;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace CoffeShopManagerment
{
    public partial class Booking : Form
    {
        int seatIndexSelected = -1; // danh sach thong tin nhung ban dang chon
        List<DTO.Menu> listMenu = new List<DTO.Menu>(); // Danh sách để chứa các sản phẩm đang chọn
        List<Product> listProduct = Product_DAO.Instance.GetListProduct();
        List<DataRow> listSeat = new List<DataRow>(); // danh sach sản phẩm
        // danh sach sản phẩm
        Product getProductData = new Product(); // bien luu tru thong tin các sản paharmsản phẩm
        Employee emp ; // nhan vien da dang login
        Button btn;

        List<DTO.Menu> listBillInfo = new List<DTO.Menu>();

        public Booking()
        {
            InitializeComponent();
            ButtonSetup.Instance.ButtonTransparent(btnDelete);
            ButtonSetup.Instance.ButtonTransparent(btnPay);
            ButtonSetup.Instance.ButtonTransparent(button1);


        }
        public Booking(Employee emp)
        {
            InitializeComponent();
            this.emp = emp;
            lblNameEmployee.Text = emp.Name;
            ButtonSetup.Instance.ButtonTransparent(btnDelete);
            ButtonSetup.Instance.ButtonTransparent(btnPay);
            ButtonSetup.Instance.ButtonTransparent(button1);


        }


        #region Method

        //Hàm load Các sản phẩm
        public void LoadProducts()
        {
            tblMovie.Controls.Clear();
            try
            {

                int row = 0;// giá trị hàng hiện tại
                foreach (var element in listProduct)
                {
                    Panel pnItemProduct = new Panel();
                    pnItemProduct.BackColor = Color.Transparent;
                    pnItemProduct.BackgroundImage = ((System.Drawing.Image)(CoffeShopManagerment.Properties.Resources.Rectangle_172));
                    pnItemProduct.BackgroundImageLayout = System.Windows.Forms.ImageLayout.Stretch;
                    pnItemProduct.Size = new System.Drawing.Size(600, 150);
                    string str = element.Name + "\nPrice:" + element.Price;
                    btn = new Button();
                    btn.FlatAppearance.BorderSize = 0;
                    btn.FlatStyle = new FlatStyle();
                    btn.BackColor = Color.Transparent;
                    btn.ForeColor = Color.White;

                    btn.Font = new System.Drawing.Font("Open Sans", 15F, System.Drawing.FontStyle.Bold);
                    btn.Text = str;
                    btn.Click += HandleProductItemClick;
                    Controls.Add(btn);
                    btn.Size = new System.Drawing.Size(200, 90);
                    this.tblMovie.RowStyles.Add(new System.Windows.Forms.RowStyle(System.Windows.Forms.SizeType.Absolute, 100));
                    btn.Tag = element;
                    btn.Location = new Point { X = 140, Y = 0 };
                    ButtonSetup.Instance.ButtonTransparent(btn);


                    Panel pn = new Panel();
                    /* Image img = Image.FromFile($"../../Resources/Images/product-{j}.png");*/
                    if (Image.FromFile(element.Image) != null){
                        Image img = Image.FromFile(element.Image);
                        pn.BackgroundImage = img;

                    }
                    pn.BackgroundImageLayout = System.Windows.Forms.ImageLayout.Zoom;
                    pn.Size = new System.Drawing.Size(80, 80);
                    pn.Location = new Point { X = 10, Y = 7 };


                    pnItemProduct.Controls.Add(pn);
                    pnItemProduct.Controls.Add(btn);

                    tblMovie.Controls.Add(pnItemProduct, 0, row++);

                }
                dateTimePicker1.Font = new Font("Arial", 18, FontStyle.Regular);
            }
            catch (Exception e) { 
                
            }
        }
        //Ham tạo các bàn
        private void LoadTables()
        {

            this.pnlSeats.Controls.Clear();

            List<Table> tables = Table_DAO.Instance.LoadTableList();
            foreach (Table item in tables)
            {
                string str = item.ID.ToString();
                Button btn = new Button();
                btn.ForeColor = Color.Black;
                btn.Padding = new Padding(10, 10, 10, 10);
                btn.FlatStyle = new FlatStyle();
                btn.FlatAppearance.BorderSize = 0;
                btn.Font = new Font("Tahoma", 12F, FontStyle.Bold);
                btn.BackgroundImage = ((System.Drawing.Image)(CoffeShopManagerment.Properties.Resources.table));
                btn.BackgroundImageLayout = System.Windows.Forms.ImageLayout.Zoom;
                btn.Text = str;
                btn.Size = new System.Drawing.Size(77, 50);
                ButtonSetup.Instance.ButtonTransparent(btn);
            
                if (item.Status == 1)
                    btn.BackColor = Color.Red;

                btn.Click += Btn_Click_Table;
                Controls.Add(btn);
                btn.Tag = item;
                this.pnlSeats.Controls.Add(btn);

            }
        }

        //hiển thị các sản phẩm trong giỏ hàng
        void ShowItemsCart(int id)
        {
            if(id != -1)
            {

                listBillInfo = Menu_DAO.Instance.GetListMenuByTable(id);
                if (listBillInfo.Count > 0 && listMenu.Count <= 0)
                {
                    tblCart.Controls.Clear();

                    long totalPrice = 0;
                    int j = 0;
                    foreach (DTO.Menu item in listBillInfo)
                    {
                        totalPrice +=item.TotalPrice;
                        Panel pnItemCard = CreateItemCard(item, j);
                        tblCart.Controls.Add(pnItemCard, 0, j);
                    }
                    lblPrice.Text = totalPrice.ToString();

                }
                else
                {

                    seatIndexSelected = id;
                }
            }



        }
        //Hàm tạo các 1 sản phẩm trong giỏ hàng
        Panel CreateItemCard(DTO.Menu menu, int j)
        {
            Panel pnItemCard = new Panel();
            pnItemCard.BackColor = Color.Transparent;
            pnItemCard.BackgroundImage = ((System.Drawing.Image)(CoffeShopManagerment.Properties.Resources.Rectangle_172));
            pnItemCard.BackgroundImageLayout = System.Windows.Forms.ImageLayout.Zoom;
            pnItemCard.Size = new System.Drawing.Size(295, 94);

            //Tạo ảnh cho sản phẩm
            Panel pnImgProduct = new Panel();
            pnImgProduct.Location = new Point { X = 10, Y = 14 };
            /*Image img = Image.FromFile($"../../Resources/Images/product-{j}.png");*/
            Image img = Image.FromFile(menu.Image);

            pnImgProduct.BackgroundImage = img;
            pnImgProduct.BackgroundImageLayout = System.Windows.Forms.ImageLayout.Zoom;
            pnImgProduct.Size = new System.Drawing.Size(65, 65);

            //Tên sản phẩm
            Label lbProductName = new Label();
            lbProductName.BackColor = Color.Transparent;
            lbProductName.Font = new System.Drawing.Font("Open Sans", 10F, System.Drawing.FontStyle.Bold);
            lbProductName.Text = $"{menu.ProductName}";
            lbProductName.ForeColor = Color.White;
            /* lbProductItem.Text = Getdatashowtimeinfo(i.Idshowtime) + "\nNumber of Seat: " + i.Numberofseats.ToString();*//**/
            lbProductName.Location = new Point { X = 81, Y = 14 };
            /*lbProductName.Size = new System.Drawing.Size(300, 28);*/
            lbProductName.MaximumSize = new Size(150, 0);
            lbProductName.AutoSize = true;

            // Giá sản phẩm
            Label lbProductPrice = new Label();
            lbProductPrice.BackColor = Color.Transparent;
            lbProductPrice.Font = new System.Drawing.Font("Open Sans", 15F, System.Drawing.FontStyle.Bold);
            lbProductPrice.Text = $"{menu.Price}";
            lbProductPrice.Location = new Point { X = 81, Y = 51 };
            lbProductPrice.Size = new System.Drawing.Size(123, 28);
            lbProductPrice.ForeColor = Color.White;

            // Số lượng sản phẩm
            Label lbProductSL = new Label();
            lbProductSL.BackColor = Color.Transparent;
            lbProductSL.Font = new System.Drawing.Font("Open Sans", 12F, System.Drawing.FontStyle.Bold);
            lbProductSL.Text = $"{menu.Count}";
            lbProductSL.Location = new Point { X = 212, Y = 37 };
            lbProductSL.ForeColor = Color.White;
            lbProductSL.Size = new System.Drawing.Size(37, 22);
            lbProductSL.TextAlign = ContentAlignment.MiddleRight;



            //Tạo 2 nút tăng và giảm
            Button btnTang = ButtonSetup.Instance.CreateButton();
            btnTang.Location = new Point { X = 229, Y = 13 };
            btnTang.BackgroundImage = ((System.Drawing.Image)(CoffeShopManagerment.Properties.Resources.plus));
            btnTang.Tag = menu;
            btnTang.Click += IncreaseQuantity;
            Button btnGiam = ButtonSetup.Instance.CreateButton();
            btnGiam.Tag = menu;
            btnGiam.Location = new Point { X = 229, Y = 60 };
            btnGiam.Click += DecreaseQuantity;

            Panel btnDelete = new Panel()
            {
                BackgroundImage = ((System.Drawing.Image)(CoffeShopManagerment.Properties.Resources.btnDeleteHCN)),
                Size = new System.Drawing.Size(37, 78),
                Location = new Point { X = 255, Y = 10 },
                BackgroundImageLayout = System.Windows.Forms.ImageLayout.Zoom,
            };
            btnDelete.Tag = menu;
            btnDelete.Click += DeleteItemCard;

            this.tblCart.RowStyles.Add(new System.Windows.Forms.RowStyle(System.Windows.Forms.SizeType.Absolute, 100));
            pnItemCard.Controls.Add(pnImgProduct);
            pnItemCard.Controls.Add(lbProductName);
            pnItemCard.Controls.Add(lbProductPrice);
            pnItemCard.Controls.Add(lbProductSL);

            pnItemCard.Controls.Add(btnTang);
            pnItemCard.Controls.Add(btnGiam);
            pnItemCard.Controls.Add(btnDelete);
            return pnItemCard;
        }


        void Btn_Click_Table(object sender, EventArgs e)
        {
            int tableID = ((sender as Button).Tag as Table).ID;
            tblCart.Tag = (sender as Button).Tag;
            if((sender as Button).BackColor == Color.Red && tblCart.Controls.Count > 0) {
                MessageBox.Show("Chỗ đã có người ngồi");
            }
            else
            {
                LoadTables();
                (sender as Button).BackColor = Color.Aqua;
                ShowItemsCart(tableID);
            }
           
        }

        //Hàm xử lý tăng số lượng của 1  sản phẩm trong giỏ hàng

        void IncreaseQuantity(object sender, EventArgs e)
        {

            DTO.Menu productBooked = ((sender as Button).Tag as DTO.Menu);
            productBooked.Count++;
                LoadItemsCart();
        }
        //Hàm xử lý giảm số lượng của 1  sản phẩm trong giỏ hàng
        void DecreaseQuantity(object sender, EventArgs e)
        {

            DTO.Menu productBooked = ((sender as Button).Tag as DTO.Menu);
            productBooked.Count = productBooked.Count > 1 ? productBooked.Count - 1 : 1;
            LoadItemsCart();
        }
        //Hàm xử lý xoá 1 sản phẩm trong giỏ hàng
        void DeleteItemCard(object sender, EventArgs e)
        {
            DTO.Menu productBooked = ((sender as Panel).Tag as DTO.Menu);

            if(listMenu.Count > 0)
            {
                listMenu.Remove(productBooked);

            }
            else if(listBillInfo.Count > 0)
            {
                listBillInfo.Remove(productBooked);
            }
            LoadItemsCart();
        }

        //Hàm Load các sản phẩm trong giỏ hàng
        void LoadItemsCart()
        {
            int row = 0;
            tblCart.Controls.Clear();
            if (listMenu.Count > 0)
            {
                int j = 0;
                long total = 0;
                foreach (DTO.Menu item in listMenu)
                {
                    total += item.Price * item.Count;
                   Panel pn = CreateItemCard(item, j++);
                    tblCart.Controls.Add(pn, 0, row++);
                }
                lblPrice.Text = total.ToString();

            }

            else if (listBillInfo.Count > 0)
            {
                tblCart.Controls.Clear();
                int j = 0;
                long total = 0;
                foreach (DTO.Menu item in listBillInfo)
                {
                    total += item.Price * item.Count;
                    Panel pn = CreateItemCard(item, j++);
                    tblCart.Controls.Add(pn, 0, row++);
                }
                lblPrice.Text = total.ToString();

            }

        }

        #endregion

        #region Events
        //Hàm xử lý thanh toán
        private void btnBooking_Click(object sender, EventArgs e)
        {

                if (seatIndexSelected != -1 && listMenu.Count > 0)
                {
                    if (MessageBox.Show("Đồng ý đặt bàn ?", "Thông báo", MessageBoxButtons.OKCancel) == System.Windows.Forms.DialogResult.OK)
                    {
                        int idBill = Math.Abs((int)DateTime.Now.ToFileTimeUtc());
                        int checkBill = Bill_DAO.Instance.InsertBill(idBill, emp.ID, seatIndexSelected, DateTime.Now.ToString(),"",0, long.Parse(lblPrice.Text));
                        int checkTable = Table_DAO.Instance.UpdateStatus(seatIndexSelected, 1);
                        int checkBillDetail = 0;
                        foreach (var item in listMenu)
                        {
                            checkBillDetail += BillDetail_DAO.Instance.InsertBillDetail(idBill, item.IDProduct, item.Count, item.Price * item.Count);
                        }
                        if (checkBill > 0 && checkTable > 0 && checkBillDetail == listMenu.Count())
                        {
                            MessageBox.Show("Đặt bàn thành công");
                        tblCart.Controls.Clear();
                            ShowItemsCart(-1);
                            LoadTables();
                        }
                        else
                        {
                            MessageBox.Show("Đã xảy ra lỗi vui lòng đặt lại");
                        }
                        listMenu.Clear();
                        seatIndexSelected = -1;
                        ShowItemsCart(-1);
                    lblPrice.Text = "0";
                    }
                }
                else if(listBillInfo.Count  > 0)
                {
                    if (MessageBox.Show("Bạn có chắc muốn thay đổi hoá đơn ?", "Thông báo", MessageBoxButtons.OKCancel) == System.Windows.Forms.DialogResult.OK)
                    {
                        int checkBill = Bill_DAO.Instance.UpdateItemBill(listBillInfo[0].ID,int.Parse(lblPrice.Text));
                        BillDetail_DAO.Instance.DeleteBillDetailByIDBill(listBillInfo[0].ID);
                        foreach (var item in listBillInfo)
                        {
                           BillDetail_DAO.Instance.InsertBillDetail(listBillInfo[0].ID, item.IDProduct, item.Count, item.Price * item.Count);

                        }
                        if(checkBill > 0)
                        {
                            tblCart.Controls.Clear();
                        lblPrice.Text = "0";

                        MessageBox.Show("Thay đổi thành công");

                        }
                    }
                }

        }


        //Sự kiện khi nhấn vào các sản phẩm
        private void HandleProductItemClick(object sender, EventArgs e)
        {

            Product product = ((sender as Button).Tag as Product);
            DTO.Menu currentMenu = listMenu.Find((DTO.Menu menu) => { return (menu.ProductName == product.Name); });
            if (currentMenu != null)
            {
                currentMenu.Count++;
            }else if(listBillInfo.Count > 0){
                listBillInfo.Add(new DTO.Menu()
                {
                    ProductName = product.Name,
                    Count = 1,
                    Price = product.Price,
                    TotalPrice = product.Price,
                    IDProduct = product.ID,
                    Image = product.Image
                });
            }
            else
            {
                listMenu.Add(new DTO.Menu()
                {
                    ProductName = product.Name,
                    Count = 1,
                    Price = product.Price,
                    TotalPrice = product.Price,
                    IDProduct = product.ID,
                    Image = product.Image
                });
            }
           
            LoadItemsCart();


        }

        private void Form3_Load(object sender, EventArgs e)
        {
            DateTime date = DateTime.Now;
            dateTimePicker1.Value.Add(date.TimeOfDay);
            LoadProducts();
            LoadTables();
        }
        //Hàm xử lý khi nhấn nút pay
        private void btnPay_Click(object sender, EventArgs e)
        {
            int blresult;
            blresult = 0;
            Table table = new Table();
            if((tblCart.Tag as Table) != null)
            {
                table = tblCart.Tag as Table;
            }
            

            int idBill = Bill_DAO.Instance.GetUncheckBillIDByTableID(table.ID);
            double totalPrice = Convert.ToInt32(lblPrice.Text);

            //MessageBox.Show(blresult.ToString());
            if (listMenu.Count > 0 || idBill >= 1)
            {
                blresult = Convert.ToInt16(MessageBox.Show("Bạn có chắc muốn thanh toán?", "Pay", MessageBoxButtons.OKCancel, MessageBoxIcon.Exclamation));
                if (blresult == 1)
                {
                    if (idBill >= 1)
                    {
                        Bill_DAO.Instance.CheckOut(idBill, (long)totalPrice);
                        Table_DAO.Instance.UpdateStatus(table.ID, 0);
                        tblCart.Controls.Clear();
                        lblPrice.Text = "0";
                        listMenu.Clear();
                        listBillInfo.Clear();

                        LoadTables();

                    }
                    else if (listMenu.Count > 0)
                    {
                        int newIdBill = Math.Abs((int)DateTime.Now.ToFileTimeUtc());
                        Bill_DAO.Instance.InsertBill(newIdBill, emp.ID, seatIndexSelected, DateTime.Now.ToString(), DateTime.Now.ToString(), 1, long.Parse(lblPrice.Text));

                        foreach (var item in listMenu)
                        {
                            BillDetail_DAO.Instance.InsertBillDetail(newIdBill, item.IDProduct, item.Count, item.Price * item.Count);

                        }
                        MessageBox.Show("Thanh toán thành công");
                        tblCart.Controls.Clear();
                        lblPrice.Text = "0";
                        listMenu.Clear();
                        listBillInfo.Clear();

                    }
                }

            }
            else
            {
                MessageBox.Show("Vui lòng chọn món rồi mới thanh toán");

            }


        }
        //Hàm xử lý khi nhấn nút xoá
        private void btnDelete_Click(object sender, EventArgs e)
        {
            tblCart.Controls.Clear();
            lblPrice.Text = "0";
            listMenu.Clear();
            listBillInfo.Clear();
            seatIndexSelected = -1;
            
        }
        //Nút X
        private void button2_Click_1(object sender, EventArgs e)
        {
            this.Dispose();
        }

        private void dateTimePicker1_ValueChanged(object sender, EventArgs e)
        {
            
            LoadProducts();
        }

        private void label3_Click(object sender, EventArgs e)
        {

        }

        private void panel9_Paint(object sender, PaintEventArgs e)
        {

        }

        private void panel1_Paint(object sender, PaintEventArgs e)
        {

        }

        private void dateTimePicker1_MouseCaptureChanged(object sender, EventArgs e)
        {
            //LoadProducts();
        }

        #endregion
    }
}
