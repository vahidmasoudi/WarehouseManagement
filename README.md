using DevExpress.XtraGrid;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Data.SqlClient;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApp2
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
            loadTable();
        }

        string connectionString = "Data Source=DESKTOP-H73189E\\MSSQLSERVER1;Initial Catalog=PioAccDx_Shahab1;User ID=ss;Password=1;Encrypt=False;";

        private void loadTable() {
    

            try
            {
                using (SqlConnection conn = new SqlConnection(connectionString))
                {
                    conn.Open();

                    string query = @"
            SELECT t.name
            FROM sys.tables t
            INNER JOIN sys.partitions p 
                ON t.object_id = p.object_id
            WHERE p.index_id IN (0,1)
            GROUP BY t.name
            HAVING SUM(p.rows) > 0
            ORDER BY t.name";

                    using (SqlCommand cmd = new SqlCommand(query, conn))
                    {
                        SqlDataReader reader = cmd.ExecuteReader();

                        comboBox1.Items.Clear();

                        while (reader.Read())
                        {
                            comboBox1.Items.Add(reader["name"].ToString());
                        }

                        reader.Close();
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }


        }
        private void button1_Click(object sender, EventArgs e)
        {


            try
            {
                string tblTable = comboBox1.Text;
                string query = "SELECT COUNT(*) FROM " + tblTable;

                using (SqlConnection conn = new SqlConnection(connectionString))
                using (SqlCommand cmd = new SqlCommand(query, conn))
                {
                    conn.Open();
                    int count = (int)cmd.ExecuteScalar();
                    MessageBox.Show($"تعداد رکوردها: {count}");
                }

            }
            catch (SqlException ex)
            {
                MessageBox.Show(
                    "خطا در اتصال به دیتابیس:\n" + ex.Message,
                    "خطای SQL",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error
                );
            }
            catch (Exception ex)
            {
                MessageBox.Show(
                    "یک خطای غیرمنتظره رخ داد:\n" + ex.Message,
                    "خطا",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error
                );
            }
        }
        private void LoadTableIntoGrid(string tableName)
        {
            if (string.IsNullOrWhiteSpace(tableName))
                return;

            tableName = tableName.Trim();
            if (!System.Text.RegularExpressions.Regex.IsMatch(tableName, @"^[A-Za-z0-9_]+$"))
            {
                MessageBox.Show("نام جدول معتبر نیست: " + tableName);
                return;
            }

            try
            {
                gridControl1.DataSource = null;

                DataTable dt = new DataTable();

                using (SqlConnection conn = new SqlConnection(connectionString))
                using (SqlCommand cmd = new SqlCommand($"SELECT * FROM [{tableName}];", conn))
                using (SqlDataAdapter da = new SqlDataAdapter(cmd))
                {
                    conn.Open();
                    da.Fill(dt);
                }

                gridControl1.DataSource = dt;
                gridView1.PopulateColumns();
                gridView1.BestFitColumns(true);
            }
            catch (Exception ex)
            {
                MessageBox.Show("خطا در نمایش داده‌ها: " + ex.Message);
            }
        }



        private void comboBox1_SelectedIndexChanged(object sender, EventArgs e)
        {
            if (comboBox1.SelectedItem == null) return;

            // اگر ComboBox فقط نام جدول داشته باشد:
            string tableName = comboBox1.SelectedItem.ToString();

            // اگر قبلاً چیزی مثل "tableName (123)" داخل comboBox می‌گذارید، اینجا باید پاکش کنید.
            // در صورت نیاز بگید تا دقیق مطابق خروجی شما تنظیم کنم.

            LoadTableIntoGrid(tableName);
        }



    }
}

