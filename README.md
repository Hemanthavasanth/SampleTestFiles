
FUNCTION Main_copy GLOBAL
    SET strEmpData TO $'''D:\\Test\\challenge.xlsx'''
    System.TerminateProcess.TerminateProcessByName ProcessName: $'''EXCEL'''
    Excel.LaunchExcel.LaunchAndOpenUnderExistingProcess Path: strEmpData Visible: False ReadOnly: False UseMachineLocale: False Instance=> ExcelInstance
    Excel.ReadFromExcel.ReadAllCells Instance: ExcelInstance GetCellContentsMode: Excel.GetCellContentsMode.PlainText FirstLineIsHeader: True RangeValue=> ExcelData
    Variables.CreateNewDatatable InputTable: { ^['Name', 'Input', 'Expires in', 'Processing notes', 'Priority', 'Unique reference', 'Status', 'Delay until'], [$'''''', $'''''', $'''''', $'''''', $'''''', $'''''', $'''''', $''''''] } DataTable=> dt_QueueItems
    Excel.CloseExcel.Close Instance: ExcelInstance
    SET intRowsCount TO ExcelData.RowsCount
    LOOP FOREACH row IN ExcelData
        SET strQueueItem TO $'''{

   \"EmpName\":\"%row['FirstName']%\",
   \"CompanyName\":\"%row['CompanyName']%\",
   \"Designation\":\"%row['RoleinCompany']%\",
   \"Email\":\"%row['Email']%\"

}'''
        @@'InputSummaryValue:WORKQUEUE': 'RPAChallenge'
DISABLE WorkQueues.EnqueueWorkQueueItem.WithoutUniqueId WorkQueue: $'''a6139a41-da34-f111-88b4-6045bdcde975''' Status: WorkQueues.WorkQueueItemEnqueueStatus.Queued Priority: WorkQueues.WorkQueueItemPriority.Normal Name: row['ID'] Value: $'''{

   \"EmpFirstName\":\"%row['FirstName']%\"
   \"LastName\":\"%row['LastName']%\"
   \"CompanyName\":\"%row['CompanyName']%\"
   \"Designation\":\"%row['RoleinCompany']%\"
   \"Email\":\"%row['Email']%\"
   \"Address\":\"%row['Address']%\"
   \"ID\":\"%row['ID']%\"
}''' WorkQueueItem=> NewWorkQueueItem
        Variables.AddRowToDataTable.AppendRowToDataTable DataTable: dt_QueueItems RowToAdd: [row['ID'], strQueueItem, '', '', 200, '', 0, '']
    END
    Variables.DeleteEmptyRowsFromDataTable DataTable: dt_QueueItems
    @@'InputSummaryValue:WORKQUEUE': 'EmployeeData'
WorkQueues.BatchEnqueueWorkQueueItems WorkQueue: $'''f0ef9346-59f7-f011-8406-6045bdcdc5d3''' WorkQueueItemData: dt_QueueItems FailedWorkQueueItems=> FailedWorkQueueItems HasFailedItems=> HasFailedItems SuccessfulWorkQueueItems=> SuccessfulWorkQueueItems
END FUNCTION




**********************************************************************************************



@@'InputSummaryValue:WORKQUEUE': 'RPAChallenge'
LOOP WHILE (WorkQueues.ProcessWorkQueueItem.ProcessWorkQueueItem WorkQueue: $'''a6139a41-da34-f111-88b4-6045bdcde975''' WorkQueueItem=> WorkQueueItem)
    Display.ShowMessageDialog.ShowMessage Title: $'''Get Workqueue Items''' Message: $'''WorkQueueItem Id - %WorkQueueItem.Id%
WorkQueue Id - %WorkQueueItem.WorkQueueId%
Name - %WorkQueueItem.Name%
Priority - %WorkQueueItem.Priority%

Value:

Value - %WorkQueueItem.Value%''' Icon: Display.Icon.Information Buttons: Display.Buttons.OK DefaultButton: Display.DefaultButton.Button1 IsTopMost: True
    WorkQueues.UpdateProcessingNotes.WithProcessingNotes WorkQueueItem: WorkQueueItem ProcessingNotes: $'''Failed because of technical exception'''
    WorkQueues.UpdateWorkQueueItem.UpdateWithProcessingNotes WorkQueueItem: WorkQueueItem Status: WorkQueues.WorkQueueItemStatus.ITException InputValue: WorkQueueItem.Id ProcessingResult: $'''Bot Run Successfully'''
END
SET strName TO $'''Michelle'''
SET FetchXML TO $'''<filter type=\"and\">
  <condition attribute=\"name\" operator=\"eq\" value=\"%strName%\"/>
</filter>
<order attribute=\"name\" descending=\"false\" />'''
@@'InputSummaryValue:WORKQUEUE': 'Test_Queue'
WorkQueues.GetWorkQueueItems WorkQueue: $'''e52a7648-9d3c-f111-88b4-6045bdcde975''' FilterRows: FetchXML RowsToReturn: 5000 WorkQueueItems=> WorkQueueItems
