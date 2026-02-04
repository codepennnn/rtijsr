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

      




     
