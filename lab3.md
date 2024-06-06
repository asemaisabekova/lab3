# Form1.cs
using System;
using System.IO;
using System.Linq;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WinFormsApp1
{
    public partial class Form1 : Form
    {
        private string? selectedFolderPath;

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            // Set form to be maximized and center screen
            this.WindowState = FormWindowState.Maximized;
            this.StartPosition = FormStartPosition.CenterScreen;

            InitializeControls();
            processFilesButton.Visible = false;
        }

        private void InitializeControls()
        {
            menuStrip1 = new MenuStrip();
            ToolStripMenuItem aboutMenuItem = new ToolStripMenuItem("About");
            aboutMenuItem.Click += AboutMenuItem_Click;
            menuStrip1.Items.Add(aboutMenuItem);
            this.Controls.Add(menuStrip1);

            folderPathTextBox = new TextBox { Location = new System.Drawing.Point(20, 40), Width = 500 };
            this.Controls.Add(folderPathTextBox);

            Button selectFolderButton = new Button { Text = "Select Folder", Location = new System.Drawing.Point(530, 40) };
            selectFolderButton.Click += SelectFolderButton_Click;
            this.Controls.Add(selectFolderButton);

            processFilesButton = new Button { Text = "Process Files", Location = new System.Drawing.Point(20, 80) };
            processFilesButton.Click += ProcessFilesButton_Click;
            this.Controls.Add(processFilesButton);

            Button additionalButton = new Button { Text = "Additional Button", Location = new System.Drawing.Point(150, 80) };
            additionalButton.Click += AdditionalButton_Click;
            this.Controls.Add(additionalButton);

            folderListBox = new ListBox { Location = new System.Drawing.Point(20, 120), Width = 300, Height = 500 };
            folderListBox.DoubleClick += FolderListBox_DoubleClick;
            this.Controls.Add(folderListBox);

            dataGridView1 = new DataGridView { Location = new System.Drawing.Point(330, 120), Width = 800, Height = 500 };
            dataGridView1.Columns.Add("Name", "File Name");
            dataGridView1.Columns.Add("LastModified", "Last Modified");
            dataGridView1.Columns.Add("Size", "File Size (KB)");
            dataGridView1.CellDoubleClick += DataGridView1_CellDoubleClick;
            this.Controls.Add(dataGridView1);
        }

        private void AboutMenuItem_Click(object sender, EventArgs e)
        {
            MessageBox.Show("Разработчик: Мевазов Камиль", "About");
        }

        private void SelectFolderButton_Click(object sender, EventArgs e)
        {
            using (FolderBrowserDialog folderBrowserDialog = new FolderBrowserDialog())
            {
                DialogResult result = folderBrowserDialog.ShowDialog();
                if (result == DialogResult.OK && !string.IsNullOrWhiteSpace(folderBrowserDialog.SelectedPath))
                {
                    selectedFolderPath = folderBrowserDialog.SelectedPath;
                    folderPathTextBox.Text = selectedFolderPath;
                    LoadFolderDataAsync(selectedFolderPath);

                    processFilesButton.Visible = true;
                }
            }
        }

        private async void LoadFolderDataAsync(string path)
        {
            folderListBox.Items.Clear();
            dataGridView1.Rows.Clear();

            string[] folders = Directory.GetDirectories(path);
            string[] files = Directory.GetFiles(path);

            foreach (string folder in folders)
            {
                folderListBox.Items.Add(Path.GetFileName(folder));
            }

            foreach (string file in files)
            {
                FileInfo fileInfo = new FileInfo(file);
                long sizeInKb = fileInfo.Length / 1024;
                dataGridView1.Rows.Add(fileInfo.Name, fileInfo.LastWriteTime, sizeInKb);
            }
        }

        private async void ProcessFilesButton_Click(object sender, EventArgs e)
        {
            Random random = new Random();
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                if (row.Cells["Name"].Value != null)
                {
                    int randomNumber = random.Next(1, dataGridView1.Rows.Count + 1);
                    await ProcessFileAsync(randomNumber);
                }
            }

            MessageBox.Show("Обработка файлов завершена.");
        }

        private void DataGridView1_CellDoubleClick(object sender, DataGridViewCellEventArgs e)
        {
            if (e.RowIndex >= 0)
            {
                var row = dataGridView1.Rows[e.RowIndex];
                if (row.Cells["Name"].Value != null)
                {
                    string fileName = row.Cells["Name"].Value.ToString();
                    string sourceFilePath = Path.Combine(selectedFolderPath, fileName);

                    DialogResult result = MessageBox.Show($"Хотите ли вы дублировать файл '{fileName}'?", "Подтверждение", MessageBoxButtons.YesNo, MessageBoxIcon.Question);

                    if (result == DialogResult.Yes)
                    {
                        using (SaveFileDialog saveFileDialog = new SaveFileDialog())
                        {
                            saveFileDialog.Filter = "All files (*.*)|*.*";
                            saveFileDialog.FileName = fileName;

                            DialogResult saveResult = saveFileDialog.ShowDialog();
                            if (saveResult == DialogResult.OK)
                            {
                                string destinationFilePath = saveFileDialog.FileName;
                                try
                                {
                                    File.Copy(sourceFilePath, destinationFilePath);
                                    MessageBox.Show($"Файл скопирован в {destinationFilePath}");
                                }
                                catch (Exception ex)
                                {
                                    MessageBox.Show($"Ошибка при копировании файла: {ex.Message}");
                                }
                            }
                        }
                    }
                }
            }
        }



        private Task ProcessFileAsync(int randomNumber)
        {
            return Task.Delay(randomNumber * 1000); // Delay in milliseconds
        }

        private void FolderListBox_DoubleClick(object sender, EventArgs e)
        {
            if (folderListBox.SelectedItem != null)
            {
                string selectedFolderName = folderListBox.SelectedItem.ToString();
                string selectedFolderFullPath = Path.Combine(selectedFolderPath, selectedFolderName);

                FolderInfoForm folderInfoForm = new FolderInfoForm(selectedFolderName, selectedFolderFullPath);
                folderInfoForm.ShowDialog();
            }
        }

        private void AdditionalButton_Click(object sender, EventArgs e)
        {
            MessageBox.Show("Па приколу");
        }
    }
}

# Form1.Designer.cs

namespace WinFormsApp1
{
    partial class Form1
    {
        private System.ComponentModel.IContainer components = null;
        private MenuStrip menuStrip1;
        private Button processFilesButton;
        private TextBox folderPathTextBox;
        private DataGridView dataGridView1;
        private ListBox folderListBox;

        private void InitializeComponent()
        {
            SuspendLayout();

            AutoScaleDimensions = new SizeF(8F, 20F);
            AutoScaleMode = AutoScaleMode.Font;
            ClientSize = new Size(1000,700) ;
            Name = "Form1";
            Text = "Form1";
            Load += Form1_Load;
            ResumeLayout(false);
        }
    }
}

# FolderInfoForm.cs

using System;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp1
{
    public partial class FolderInfoForm : Form
    {
        private string folderName;
        private string folderFullPath;

        public FolderInfoForm(string folderName, string folderFullPath)
        {
            InitializeComponent();
            this.folderName = folderName;
            this.folderFullPath = folderFullPath;

            folderNameLabel.Text = $"Folder Name: {folderName}";
            folderFullPathLabel.Text = $"Folder Path: {folderFullPath}";
        }

        private void InitializeComponent()
        {
            this.folderNameLabel = new System.Windows.Forms.Label();
            this.folderFullPathLabel = new System.Windows.Forms.Label();
            this.SuspendLayout();
            //
            // 
            // folderNameLabel
            // 
            this.folderNameLabel.AutoSize = true;
            this.folderNameLabel.Location = new System.Drawing.Point(13, 13);
            this.folderNameLabel.Name = "folderNameLabel";
            this.folderNameLabel.Size = new System.Drawing.Size(93, 20);
            this.folderNameLabel.TabIndex = 0;
            this.folderNameLabel.Text = "Folder Name";
            // 
            // folderFullPathLabel
            // 
            this.folderFullPathLabel.AutoSize = true;
            this.folderFullPathLabel.Location = new System.Drawing.Point(13, 43);
            this.folderFullPathLabel.Name = "folderFullPathLabel";
            this.folderFullPathLabel.Size = new System.Drawing.Size(89, 20);
            this.folderFullPathLabel.TabIndex = 1;
            this.folderFullPathLabel.Text = "Folder Path";
            // 
            // FolderInfoForm
            // 
            this.ClientSize = new System.Drawing.Size(282, 253);
            this.Controls.Add(this.folderFullPathLabel);
            this.Controls.Add(this.folderNameLabel);
            this.Name = "FolderInfoForm";
            this.ResumeLayout(false);
            this.PerformLayout();
        }

        private System.Windows.Forms.Label folderNameLabel;
        private System.Windows.Forms.Label folderFullPathLabel;
    }
}


# FileInfoForm.cs

using System;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp1
{
    public partial class FileInfoForm : Form
    {
        private string fileName;
        private string fileLastModified;
        private string fileSize;

        public FileInfoForm(string fileName, string fileLastModified, string fileSize)
        {
            InitializeComponent();
            this.fileName = fileName;
            this.fileLastModified = fileLastModified;
            this.fileSize = fileSize;

            fileNameLabel.Text = $"File Name: {fileName}";
            fileLastModifiedLabel.Text = $"Last Modified: {fileLastModified}";
            fileSizeLabel.Text = $"File Size: {fileSize} KB";
        }

        private void InitializeComponent()
        {
            this.fileNameLabel = new System.Windows.Forms.Label();
            this.fileLastModifiedLabel = new System.Windows.Forms.Label();
            this.fileSizeLabel = new System.Windows.Forms.Label();
            this.SuspendLayout();
            // 
            // fileNameLabel
            // 
            this.fileNameLabel.AutoSize = true;
            this.fileNameLabel.Location = new System.Drawing.Point(13, 13);
            this.fileNameLabel.Name = "fileNameLabel";
            this.fileNameLabel.Size = new System.Drawing.Size(72, 20);
            this.fileNameLabel.TabIndex = 0;
            this.fileNameLabel.Text = "File Name";
            // 
            // fileLastModifiedLabel
            // 
            this.fileLastModifiedLabel.AutoSize = true;
            this.fileLastModifiedLabel.Location = new System.Drawing.Point(13, 43);
            this.fileLastModifiedLabel.Name = "fileLastModifiedLabel";
            this.fileLastModifiedLabel.Size = new System.Drawing.Size(100, 20);
            this.fileLastModifiedLabel.TabIndex = 1;
            this.fileLastModifiedLabel.Text = "Last Modified";
            // 
            // fileSizeLabel
            // 
            this.fileSizeLabel.AutoSize = true;
            this.fileSizeLabel.Location = new System.Drawing.Point(13, 73);
            this.fileSizeLabel.Name = "fileSizeLabel";
            this.fileSizeLabel.Size = new System.Drawing.Size(66, 20);
            this.fileSizeLabel.TabIndex = 2;
            this.fileSizeLabel.Text = "File Size";
            // 
            // FileInfoForm
            // 
            this.ClientSize = new System.Drawing.Size(282, 253);
            this.Controls.Add(this.fileSizeLabel);
            this.Controls.Add(this.fileLastModifiedLabel);
            this.Controls.Add(this.fileNameLabel);
            this.Name = "FileInfoForm";
            this.ResumeLayout(false);
            this.PerformLayout();
        }

        private System.Windows.Forms.Label fileNameLabel;
        private System.Windows.Forms.Label fileLastModifiedLabel;
        private System.Windows.Forms.Label fileSizeLabel;
    }
}
