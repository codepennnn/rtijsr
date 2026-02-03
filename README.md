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

// Helper: sanitize a file name by removing invalid filename chars and collapsing multiple dashes/underscores.
// keeps the extension intact.
private static string MakeSafeFileName(string fileName)
{
    if (string.IsNullOrEmpty(fileName))
        return fileName;

    var invalidChars = Path.GetInvalidFileNameChars();
    var name = Path.GetFileNameWithoutExtension(fileName);
    var ext = Path.GetExtension(fileName);

    // Replace invalid chars with underscore
    var cleaned = new StringBuilder();
    foreach (var ch in name)
    {
        cleaned.Append(invalidChars.Contains(ch) ? '_' : ch);
    }

    // Collapse runs of underscores
    var result = System.Text.RegularExpressions.Regex.Replace(cleaned.ToString(), "_{2,}", "_").Trim('_');

    // Optionally trim length
    var maxBaseLength = 100;
    if (result.Length > maxBaseLength)
        result = result.Substring(0, maxBaseLength);

    return result + ext;
}



// Helper: if the candidate file already exists, add a numeric suffix before the extension.
private static string GetUniqueFileName(string folderPath, string candidateFileName)
{
    var baseName = Path.GetFileNameWithoutExtension(candidateFileName);
    var ext = Path.GetExtension(candidateFileName);
    var attempt = 0;
    var fileName = candidateFileName;
    while (File.Exists(Path.Combine(folderPath, fileName)))
    {
        attempt++;
        fileName = $"{baseName}_{attempt}{ext}";
    }
    return fileName;
}
