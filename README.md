# C-_projectusing Aprovizionare3.Extensions;
using Aprovizionare3.Repositories;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Runtime.Serialization.Formatters.Binary;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace Aprovizionare3
{
    public partial class Local : Form
    {

        private readonly IRepository _Repository;
        private string _mode;
        public Local(string mode)
        {
            InitializeComponent();

            _Repository = new Repository();
            _mode = mode;
            dataGridView1.AutoGenerateColumns = false;

            if (mode == "eventCheckBox")
            {
                
                titleTextBox.Text = "Mode of supply";
                appetizersTextBox.Text = _Repository.GetEventMenu().App;
                firstDishTextBox.Text = _Repository.GetEventMenu().F;
                secondDishTextBox.Text = _Repository.GetEventMenu().S;
                desertTextBox.Text = _Repository.GetEventMenu().D;
                drinksTextBox.Text = _Repository.GetEventMenu().D_;
                this.Text = "Supply";   
                this.Size = new Size(745, 768);
                dataGridView1.Size = new Size(370, 185);

                dataGridView1
                .AddColumn(0, "Order Number", "TableNumber")
                
               
                .AddButtonColumn("deleteColumn", "Clear Table", "Clear");

                dataGridView1.CellClick += dataGridView1_CellClick;

                //dataGridView1.DataSource = _restaurantRepository.GetAllEventTable();

            }

            if (mode == "clientCheckBox")
            {
                
                addTableButton.Text = "Add order";
                titleTextBox.Text = "Clients";
                groupBox1.Visible = false;
                this.Text = "Clients";
                this.Size = new Size(1300, 350);
                dataGridView1.Size = new Size(1200, 185);

                //comanda pentru mai multi clienti daca e cazul de anumite lucruri
                dataGridView1
                .AddColumn(0, "Order Number", "TableNumber")
                .AddColumn(1, "Customer name for order", "ReservationName")
                .AddColumn(2, "Reservation", "Reservation")
                .AddColumn(3, "Date", "DateTime")
                
                .AddColumn(4, "Clients Number", "ClientsNumber")
                .AddColumn(5, "Total", "Total")
                .AddButtonColumn("Order", "Menu", "View Table Order")
                .AddButtonColumn("editColumn", "Change  Info", "Edit")
                .AddButtonColumn("deleteColumn", "Clear Table", "Clear");


                dataGridView1.CellClick += dataGridView1_CellClick;


            }


        }

        private void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
        {
            if (_mode == "clientCheckBox")
            {
                if (e.ColumnIndex == dataGridView1.Columns["editColumn"].Index)
                {
                    var entity = dataGridView1.CurrentRow.DataBoundItem as Table;
                    var editTableForm = new EditTableForm(entity.Id, _mode);
                    editTableForm.ShowDialog();
                    var data = _Repository.GetAllNormalTable();
                    dataGridView1.RefreshDataGrid(data);
                }
            }

            if (e.ColumnIndex == dataGridView1.Columns["deleteColumn"].Index)
            {
                var entity = (Table)dataGridView1.CurrentRow.DataBoundItem as Table;
                _Repository.DeleteTable(entity);
                if (_mode == "clientCheckBox")
                {
                    var data = _Repository.GetAllNormalTable();
                    dataGridView1.RefreshDataGrid(data);
                }
                else
                {
                    var data = _Repository.GetAllEventTable();
                    dataGridView1.RefreshDataGrid(data);
                }
            }


            if (_mode == "clientCheckBox")
            {
                if (e.ColumnIndex == dataGridView1.Columns["order"].Index)
                {
                    var menuForm = new Menu();
                    this.Hide();
                    menuForm.ShowDialog();
                    this.Show();
                    var entity = (Table)dataGridView1.CurrentRow.DataBoundItem as Table;
                    entity.Total = menuForm.total;

                }
            }
        }
       
        private void addTableButton_Click(object sender, EventArgs e)
        {
           
            if (_mode == "clientCheckBox")
            {
                var addTableForm = new AddTableForm();
                addTableForm.ShowDialog();
                var data = _Repository.GetAllNormalTable();
                dataGridView1.RefreshDataGrid(data);
            }
            else
            {
                var table = new Table();
                table.ClientsNumber = 0;
                table.Event = true;
                _Repository.AddTable(table);
                var data = _Repository.GetAllEventTable();
                dataGridView1.RefreshDataGrid(data);
            }
            
        }
        private void exportButton_Click(object sender, EventArgs e)
        {
            dataGridView1.ExportNormalTableData(_mode + ".dat", _mode);
        }

        private void importButton_Click(object sender, EventArgs e)
        {
            dataGridView1.ImportNormalTableData(_mode + ".dat", _mode);
        }

        private void Local_Load(object sender, EventArgs e)
        {

        }

        private void groupBox1_Enter(object sender, EventArgs e)
        {

        }

        private void appetizersTextBox_TextChanged(object sender, EventArgs e)
        {

        }
    }
}
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Aprovizionare3
{
    public static class FakeDatabase
    {
        
        public static List<Person> GuestList = new List<Person>();

        public static List<Person> ClientsList = new List<Person>();

        public static Menu1 Menu1 = new Menu1()
        {
            App = "Calculatoare și laptopuri",
            F = "Sisteme de operare (Windows, macOS, Linux)",
            S = "Switch-uri și routere",
            D = "Sisteme de stocare în cloud",
            D_ = "Instalare și configurare echipamente și software"
        };

        public static List<Table> Tables = new List<Table>();
    }
}
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Aprovizionare3
{
    [Serializable]
    public class Table
    {
        public static int nr = 1;

        public Guid Id { get; set; }
        public int TableNumber { get; set; }
        public int ClientsNumber { get; set; }
        public bool Reservation { get; set; }
        public DateTime DateTime { get; set; }
        public string Serving { get; set; }
        public double Total { get; set; }
        public bool Event = false;
        public string ReservationName { get; set; }

        public Table()
        {
            TableNumber = nr++;
        }

        public bool Equals(Table table)
        {
            if (table == null) return false;

            return Id == table.Id;
        }
    }
}

using Aprovizionare3.Repositories;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace Aprovizionare3
{
    public partial class EditTableForm : Form
    {
        private readonly IRepository _Repository;
        private Guid _id;
        private string _mode;
        public EditTableForm(Guid id, string mode)
        {
            InitializeComponent();
            _Repository = new Repository();
            _id = id;
            _mode = mode;
            if (mode == "eventCheckBox")
            {
                label1.Visible = false;
                label3.Visible = false;
                editTotalTextBox.Visible = false;
            }
        }

        private void editButton_Click(object sender, EventArgs e)
        {
            Validator.Validate(Controls, errorProvider1);

            foreach (Control control in Controls)
            {
                if (errorProvider1.GetError(control).ToString().Length > 1)
                    return;
            }

            var table = _Repository.GetTableById(_id);
            table.Serving = servingComboBox.SelectedItem.ToString();
            table.ClientsNumber = Int32.Parse(editClientsNumberTextBox.Text);
            table.DateTime = new DateTime(dateTimePicker1.Value.Year,
                    dateTimePicker1.Value.Month,
                    dateTimePicker1.Value.Day,
                    hour: Int32.Parse(hourTextBox.Text),
                    minute: Int32.Parse(minuteTextBox.Text)
                    , 0);
            table.Total = Int32.Parse(editTotalTextBox.Text);

            this.Hide();
        }

        private void editTotalTextBox_KeyPress(object sender, KeyPressEventArgs e)
        {
            if (e.KeyChar == (char)Keys.Enter)
            {
                e.Handled = true;
                editButton_Click(sender, null);
            }
        }

        private void EditTableForm_Load(object sender, EventArgs e)
        {

        }

        private void label5_Click(object sender, EventArgs e)
        {

        }
    }
}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Aprovizionare3
{
    [Serializable]
    public class Person
    {
        public Guid Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string PhoneNumber { get; set; }
        public Guid TableId { get; set; }
        //public string Email { get; set; }
    }
}
