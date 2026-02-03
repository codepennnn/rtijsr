  protected void btnSave_Click(object sender, EventArgs e)
  {           
      bocw_year_end.UnbindData();
      if (Assessment_filename.HasFile)
      {
          string getFileName = "";
          List<string> FileList = new List<string>();
          foreach (HttpPostedFile htfiles in Assessment_filename.PostedFiles)
          {
              getFileName = PageRecordDataSet.Tables["App_BOCW_Yearly"].Rows[0]["ID"].ToString() + "_" + Path.GetFileName(htfiles.FileName);
              getFileName = Regex.Replace(getFileName, @"[,+*/?|><&=\#%:;@[^$?:'()!~}{`]", "");
              htfiles.SaveAs((@"D:/Cybersoft_Doc/CLMS/Attachments/" + getFileName));

              FileList.Add(getFileName);
              getFileName = "";
          }
          PageRecordDataSet.Tables["App_BOCW_Yearly"].Rows[0]["Assessment_filename"] = string.Join(",", FileList);
      }




      protected void btnSave_Click(object sender, EventArgs e)
{
    bocw_year_end.UnbindData();

    if (!Assessment_filename.HasFile)
        return;

    // Get ID from dataset (validate existence in real code)
    var id = PageRecordDataSet.Tables["App_BOCW_Yearly"].Rows[0]["ID"].ToString();

    // Get upload folder from config, fallback to ~/Attachments
    var uploadFolderSetting = System.Configuration.ConfigurationManager.AppSettings["UploadPath"];
    var uploadFolder = !string.IsNullOrWhiteSpace(uploadFolderSetting)
        ? uploadFolderSetting
        : Server.MapPath("~/Attachments");

    // Ensure directory exists
    if (!Directory.Exists(uploadFolder))
        Directory.CreateDirectory(uploadFolder);

    var savedFileNames = new List<string>();

    // Example of allowed extensions and max size (bytes). Adjust as needed.
    var allowedExtensions = new HashSet<string>(StringComparer.OrdinalIgnoreCase) { ".pdf", ".docx", ".doc", ".xlsx", ".jpg", ".png" };
    const int maxFileSizeBytes = 25 * 1024 * 1024; // 25 MB

    foreach (HttpPostedFile postedFile in Assessment_filename.PostedFiles)
    {
        try
        {
            if (postedFile == null || postedFile.ContentLength == 0)
                continue;

            if (postedFile.ContentLength > maxFileSizeBytes)
            {
                // handle too-large file (log or notify user)
                continue;
            }

            var originalFileName = Path.GetFileName(postedFile.FileName);
            var extension = Path.GetExtension(originalFileName);

            if (string.IsNullOrEmpty(extension) || !allowedExtensions.Contains(extension))
            {
                // handle disallowed extension (log or notify user)
                continue;
            }

            var safeName = MakeSafeFileName(originalFileName);

            // Prefix with ID to keep relation and prevent name collisions in multi-record environment.
            var candidateName = $"{id}_{safeName}";
            var uniqueName = GetUniqueFileName(uploadFolder, candidateName);
            var fullPath = Path.Combine(uploadFolder, uniqueName);

            postedFile.SaveAs(fullPath);
            savedFileNames.Add(uniqueName);
        }
        catch (Exception ex)
        {
            // TODO: log error (ex)
            // optionally inform user about failure to save this file
        }
    }

    if (savedFileNames.Any())
    {
        PageRecordDataSet.Tables["App_BOCW_Yearly"].Rows[0]["Assessment_filename"] = string.Join(",", savedFileNames);
    }
}

