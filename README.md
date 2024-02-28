# Business

   public ChannelCloseOpen(ExecutionDataContext context) : base(context)
        {
        }

        /// <summary>
        /// Kulllanıcı kanal listesi
        /// </summary>
        /// <param name="userId"></param>
        /// <returns></returns>
        public GenericResponse<List<ChannelCloseOpenContract>> GetUserChannelList(int userId)
        {
            SqlCommand command;
            GenericResponse<List<ChannelCloseOpenContract>> returnObject;
            GenericResponse<SqlDataReader> sp;
            returnObject = this.InitializeGenericResponse<List<ChannelCloseOpenContract>>("BOA.Business.InternetBanking.UserSettings.ChannelCloseOpen.GetUserChannelList");
            command = this.DBLayer.GetDBCommand(Databases.BoaWeb, "INT.sel_UserChannelListByUserId");
            // Parameters
            this.DBLayer.AddInParameter(command, "@UserID", SqlDbType.Int, userId);
            sp = this.DBLayer.ExecuteReader(command);
            if (!sp.Success)
            {
                returnObject.Results.AddRange(sp.Results);
                if (sp.Value != null)
                    sp.Value.Close();
                return returnObject;
            }
            #region Fill from SqlDataReader to List
            List<ChannelCloseOpenContract> listOfDataContract = new List<ChannelCloseOpenContract>();
            ChannelCloseOpenContract dataContract = null;
            SqlDataReader reader = sp.Value;
            while (reader.Read())
            {
                dataContract = new ChannelCloseOpenContract();
                dataContract.UserChannelId = SQLDBHelper.GetInt32Value(reader["UserChannelId"]);
                dataContract.UserId = SQLDBHelper.GetInt32Value(reader["UserId"]);
                dataContract.ChannelId = SQLDBHelper.GetByteValue(reader["ChannelId"]);
                dataContract.ChannelName = SQLDBHelper.GetStringValue(reader["ChannelName"]);
                listOfDataContract.Add(dataContract);
            }
            reader.Close();
            //Return 
            returnObject.Value = listOfDataContract;
            #endregion
            return returnObject;
        }

        /// <summary>
        /// Kanal liste insert
        /// </summary>
        /// <param name="contract"></param>
        /// <returns></returns>
        public GenericResponse<Int32> Insert(ChannelCloseOpenContract contract)
        {
            SqlCommand command;
            GenericResponse<Int32> returnObject;
            GenericResponse<Int32> sp;
            returnObject = this.InitializeGenericResponse<Int32>("BOA.Business.InternetBanking.UserSettings.ChannelCloseOpen.Insert");
            if (contract == null)
            {
                returnObject.Results.Add(new Result()
                {
                    ErrorMessage = BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "ParameterNullError"),
                    Severity = Severity.Error
                });
                return returnObject;
            }
            command = this.DBLayer.GetDBCommand(Databases.BoaWeb, "INT.ins_UserChannel");
            // Parameters
            this.DBLayer.AddInParameter(command, "ChannelId", SqlDbType.VarChar, contract.ChannelId);
            this.DBLayer.AddInParameter(command, "UserId", SqlDbType.Bit, contract.UserId);
            sp = this.DBLayer.ExecuteScalar<Int32>(command);
            if (!sp.Success)
            {
                returnObject.Results.AddRange(sp.Results);
                return returnObject;
            }
            //Return 
            contract.UserChannelId = sp.Value;
            returnObject.Value = sp.Value;
            return returnObject;
        }

        /// <summary>
        /// Kanal sil
        /// </summary>
        /// <param name="userChannelId"></param>
        /// <returns></returns>
        public GenericResponse<int> Delete(int userChannelId)
        {
            SqlCommand command;
            GenericResponse<int> returnObject;
            GenericResponse<int> sp;
            returnObject = this.InitializeGenericResponse<int>("BOA.Business.InternetBanking.UserSettings.ChannelCloseOpen.Delete");
            command = this.DBLayer.GetDBCommand(Databases.BoaWeb, "INT.del_UserChannel");
            // Parameters
            this.DBLayer.AddInParameter(command, "UserChannelId", SqlDbType.Int, userChannelId);
            sp = this.DBLayer.ExecuteNonQuery(command);
            if (!sp.Success)
            {
                returnObject.Results.AddRange(sp.Results);
                return returnObject;
            }
            returnObject.Value = sp.Value;
            return returnObject;
        }



    }
}


orchestration:


using BOA.Base;
using BOA.Common.Types;
using BOA.Types.InternetBanking;
using BOA.Types.InternetBanking.UserSettings;
using BOA.Types.Kernel.InternetBanking;
using System.Collections.Generic;
using System.Linq;

namespace BOA.Orchestration.InternetBanking.UserSettings
{

    public class ChannelCloseOpen
    {

        /// <summary>
        /// Kanal listesi
        /// </summary>
        /// <param name="request"></param>
        /// <param name="objectHelper"></param>
        /// <returns></returns>
        public GenericResponse<List<ChannelCloseOpenContract>> GetUserChannelList(ChannelCloseOpenRequest request, ObjectHelper objectHelper)
        {
            GenericResponse<List<ChannelCloseOpenContract>> returnObject = objectHelper.InitializeGenericResponse<List<ChannelCloseOpenContract>>("BOA.Orchestration.InternetBanking.UserSettings.ChannelCloseOpen.GetUserChannelList");
            BOA.Business.InternetBanking.UserSettings.ChannelCloseOpen bo = new BOA.Business.InternetBanking.UserSettings.ChannelCloseOpen(objectHelper.Context);
            GenericResponse<List<BOA.Types.InternetBanking.UserSettings.ChannelCloseOpenContract>> responseChannel = bo.GetUserChannelList(request.UserId);
            if (!responseChannel.Success)
            {
                returnObject.Results.AddRange(responseChannel.Results);
                return returnObject;
            }
            returnObject.Value = responseChannel.Value;
            return returnObject;
        }

        /// <summary>
        /// Kullanıcı kullandığı kanalı silme
        /// </summary>
        /// <param name="request"></param>
        /// <param name="objectHelper"></param>
        /// <returns></returns>
        public WorkflowResponse<bool> DeleteAndCloseUserChannel(ChannelCloseOpenRequest request, ObjectHelper objectHelper)
        {
            var returnObject = objectHelper.InitializeResponse<WorkflowResponse<bool>>("BOA.Orchestration.InternetBanking.UserSettings.ChannelCloseOpen.DeleteAndCloseUserChannel");
            var process = new Process.Kernel.InternetBanking.WebUser();

            var webUserChannelContract = new WebUserChannelContract
            {
                BusinessKey = request.BusinessKey,
                Description = request.Description,
                Reason = (CloseReasonEnum)request.ReasonId,
                UserId = request.UserId,
                SelectedNoteId = request.SelectedNoteId
            };

            foreach (ChannelCloseOpenContract item in request.UserChannelList)
            {
                webUserChannelContract.UserChannelList.Add((ChannelContract)item.ChannelId);
            }

            var responseProcess = process.DeleteUserChannels(webUserChannelContract, objectHelper);
            if (!responseProcess.Success)
            {
                returnObject.Results.AddRange(responseProcess.Results);
                return returnObject;
            }
            returnObject.Value = true;
            return returnObject;
        }

        /// <summary>
        /// Kullanıcı kanal ekleme
        /// </summary>
        /// <param name="request"></param>
        /// <param name="objectHelper"></param>
        /// <returns></returns>
        public WorkflowResponse<bool> AddUserChannel(ChannelCloseOpenRequest request, ObjectHelper objectHelper)
        {
            WorkflowResponse<bool> returnObject = objectHelper.InitializeResponse<WorkflowResponse<bool>>("BOA.Orchestration.InternetBanking.UserSettings.ChannelCloseOpen.AddUserChannel");
            BOA.Process.InternetBanking.Login.CustomerProcess boBackOfficeAddUserChannel = new BOA.Process.InternetBanking.Login.CustomerProcess();

            var response = boBackOfficeAddUserChannel.AddUserChannel(request.UserId, request.RoleId, request.UserChannelList.Select(p => (int)p.ChannelId).ToList(), request.UserCode, request.BusinessKey, objectHelper);
            if (response.Success == false || response.Results.Count > 0)
            {
                returnObject.Results.AddRange(response.Results);
                return returnObject;
            }
            returnObject.Value = true;
            return returnObject;
        }

        /// <summary>
        /// Kanal listesi
        /// </summary>
        /// <param name="request"></param>
        /// <param name="objectHelper"></param>
        /// <returns></returns>
        public GenericResponse<List<UserChannelHistoryContract>> SelectUserChannelHistoryList(ChannelCloseOpenRequest request, ObjectHelper objectHelper)
        {
            GenericResponse<List<UserChannelHistoryContract>> returnObject =
                    objectHelper.InitializeGenericResponse<List<UserChannelHistoryContract>>("BOA.Orchestration.InternetBanking.UserSettings.ChannelCloseOpen.SelectUserChannelHistoryList");
            BOA.Business.InternetBanking.UserChannelHistory boUserChannel = new Business.InternetBanking.UserChannelHistory(objectHelper.Context);
            GenericResponse<List<UserChannelHistoryContract>> responseChannel = boUserChannel.GetUserChannelHistoryListByUserId(request.UserId);
            if (!responseChannel.Success)
            {
                returnObject.Results.AddRange(responseChannel.Results);
                return returnObject;
            }
            returnObject.Value = responseChannel.Value;
            return returnObject;
        }
    }
}



Types:

using BOA.Common.Types;
using System;
using System.Collections.Generic;

namespace BOA.Types.InternetBanking.UserSettings
{
    [Serializable]
    public class ChannelCloseOpenRequest : RequestBase
    {
        //public int UserChannelId { get; set; }
        //public int UserId { get; set; }
        public bool IsAllChannel { get; set; }





        public ChannelCloseOpenRequest()
        {
            WorkFlowData = new WorkFlowRequestData();
            WorkFlowInternalData = new WorkFlowRequestInternalData();
        }
        public WorkFlowRequestData WorkFlowData
        {
            get;
            set;
        }

        public WorkFlowRequestInternalData WorkFlowInternalData
        {
            get;
            set;
        }
        public int UserId { get; set; }

        public byte ChannelId { get; set; }

        public IEnumerable<ChannelCloseOpenContract> SelectUserChannelList { get; set; }
        public List<UserChannelHistoryContract> SelectUserChannelHistoryList { get; set; }
        public List<ChannelCloseOpenContract> UserChannelList = new List<ChannelCloseOpenContract>();

        public int UserStateTypeId { get; set; }

        public int RoleId { get; set; }

        public string UserCode { get; set; }

        public string Description { get; set; }

        public int ReasonId { get; set; }

        public string DivitInstanceId
        {
            get;
            set;
        }

        public int SelectedNoteId
        {
            get;
            set;
        }

    }
}


Types Contract:

using BOA.Common.Types;
using System;

namespace BOA.Types.InternetBanking.UserSettings
{
    [Serializable]
    public class ChannelCloseOpenContract : ContractBase
    {
        public byte ChannelId { get; set; }
        public string ChannelName { get; set; }
        public int UserChannelId { get; set; }

        public int UserId { get; set; }

        public bool isReadOnly = false;

        public string description;

        public int reasonId;


    }
}


controller:

using BOA.Common.Types;
using BOA.Types.InternetBanking;
using BOA.Types.InternetBanking.UserSettings;
using BOA.Web.Base;
using BOA.Web.Base.Types;
using BOA.Web.InternetBanking.ChannelCloseOpen.Models;
using System.Collections.Generic;
using System.Linq;
using System.Web.Mvc;

namespace BOA.Web.InternetBanking.ChannelCloseOpen.Controllers
{

    public partial class ChannelCloseOpenController : BTransactionalWizardController
    {
        #region PrepareData

        private BActionResult<BWizardModel> PrepareIndexData()
        {
            var returnObject = new BActionResult<BWizardModel>();
            bool IsAllChannelTrue;
            TransactionContext.CurrentTransactionCode = TransactionCode.CHANNEL_CLOSE_OPEN;
            var indexModel = GetModel<IndexModel>(IndexView.Name);
            if (indexModel == null)
            {
                indexModel = new IndexModel();
            }

            if (indexModel.OpenInternet == "1" && indexModel.OpenMobile == "1" && indexModel.OpenTelephone == "1" && indexModel.OpenApi == "1")
            {
                IsAllChannelTrue = true;
            }
            else
            {
                IsAllChannelTrue = false;
            }

            WebUserContract webUser = WebContext.UserDataDictionary[BOA.Web.InternetBanking.Types.SessionKeys.WebUser] as WebUserContract;

            ChannelCloseOpenRequest ChannelRequest = new ChannelCloseOpenRequest();
            var requestGetChannel = new ChannelCloseOpenRequest
            {
                UserId = webUser.CustomerPersonId,
                MethodName = "GetUserChannelList",
                IsAllChannel = IsAllChannelTrue
            };

            var serviceResponseGetChannelCloseOpen = Execute<ChannelCloseOpenRequest, GenericResponse<List<ChannelCloseOpenContract>>>(requestGetChannel, true);
            if (!serviceResponseGetChannelCloseOpen.Success)
            {
                returnObject.AddMessage(BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "AccountsParticipationAccountMergeGeneralErrorMessage", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), MessageType.Error, serviceResponseGetChannelCloseOpen.Results);
                return returnObject;
            }

            if (serviceResponseGetChannelCloseOpen.Value != null && serviceResponseGetChannelCloseOpen.Value.Count > 0)
            {

                var Channel = serviceResponseGetChannelCloseOpen.Value.Where(item => item.ChannelId == 41 || item.ChannelId == 2 || item.ChannelId == 5 || item.ChannelId == 6).FirstOrDefault();

                if (Channel != null)
                {
                    indexModel.OpenInternetValue = 1;
                    indexModel.OpenTelephoneValue = 1;
                    indexModel.OpenMobileValue = 1;
                    indexModel.OpenApiValue = 1;
                    indexModel.OpenTelephone = "1";
                    indexModel.OpenInternet = "1";
                    indexModel.OpenMobile = "1";
                    indexModel.OpenApi = "1";


                }
                else if (Channel == null)
                {
                    indexModel.OpenInternetValue = 0;
                    indexModel.OpenInternetValue = 0;
                    indexModel.OpenTelephoneValue = 0;
                    indexModel.OpenMobileValue = 0;
                    indexModel.OpenApiValue = 0;
                    indexModel.OpenTelephone = "0";
                    indexModel.OpenInternet = "0";
                    indexModel.OpenMobile = "0";
                    indexModel.OpenApi = "0";


                }



            }
            returnObject.Model = indexModel;

            return returnObject;

        }
















        #endregion

        #region Navigate

        [HttpPost]
        public ActionResult Index(IndexModel model)
        {
            return Navigate(model);
        }

        #endregion

        #region BeforeValidate

        private BActionResult<BWizardModel> BeforeValidateIndexData(BWizardModel model)
        {
            var returnObject = new BActionResult<BWizardModel>();

            var indexModel = model as IndexModel;

            if (indexModel.OpenInternet == "0")
            {
                indexModel.OpenInternetValue = 0;
            }
            else if (indexModel.OpenInternet == "1")
            {
                indexModel.OpenInternetValue = 1;
            }

            if (indexModel.OpenMobile == "0")
            {
                indexModel.OpenMobileValue = 0;
            }
            else if (indexModel.OpenMobile == "1")
            {
                indexModel.OpenMobileValue = 1;
            }

            if (indexModel.OpenTelephone == "0")
            {
                indexModel.OpenTelephoneValue = 0;
            }
            else if (indexModel.OpenTelephone == "1")
            {
                indexModel.OpenTelephoneValue = 1;
            }

            if (indexModel.OpenApi == "1")
            {
                indexModel.OpenApiValue = 1;
            }

            else if (indexModel.OpenApi == "0")
            {
                indexModel.OpenApiValue = 0;
            }

            returnObject.Model = indexModel;

            return returnObject;


        }

        #endregion

        #region Validate

        private BActionResult ValidateIndexData(BWizardModel model)
        {
            var returnObject = new BActionResult();

            var indexModel = GetModel<IndexModel>(IndexView.Name);

            return returnObject;
        }

        #endregion
    }
}

closecontroller:

using BOA.Common.Types;
using BOA.Types.InternetBanking;
using BOA.Types.InternetBanking.UserSettings;
using BOA.Web.Base;
using BOA.Web.Base.Types;
using BOA.Web.InternetBanking.ChannelCloseOpen.Models;
using static BOA.Types.InternetBanking.UserSettings.ChannelCloseOpenRequest;

namespace BOA.Web.InternetBanking.ChannelCloseOpen.Controllers
{
    public partial class ChannelCloseOpenController : BTransactionalWizardController
    {
        #region View Definitions

        private BView ConfirmView = null;

        protected override void InitializeViews()
        {
            IndexView.PrepareData = PrepareIndexData;
            IndexView.BeforeValidate = BeforeValidateIndexData;
            IndexView.Validate = ValidateIndexData;
            DisplayResultView.PrepareDisplayResultData = PrepareDisplayResultData;

            ConfirmView = new BView("Confirm");
            ConfirmView.PrepareData = PrepareConfirmData;
            ConfirmView.Validate = ValidateConfirmData;
            ViewList.Add(ConfirmView);
        }

        protected override BView GetNextView(string fromStep)
        {
            if (fromStep == IndexView.Name)
            {
                return ConfirmView;
            }

            return IndexView;
        }

        #endregion

        #region Execute

        protected override BActionResult<BWizardModel> ExecuteAction()
        {
            var returnObject = new BActionResult<BWizardModel>();

            var indexModel = GetModel<IndexModel>(IndexView.Name);

            returnObject.Model = new DisplayResultModel();


            WebUserContract webUser = WebContext.UserDataDictionary[BOA.Web.InternetBanking.Types.SessionKeys.WebUser] as WebUserContract;

            ChannelCloseOpenRequest ChannelCloseOpenRequest = new ChannelCloseOpenRequest();
            var requestCloseOpenChannel = new ChannelCloseOpenRequest
            {
                UserId = webUser.CustomerPersonId,
                MethodName = "DeleteAndCloseUserChannel",

            };
            var responseSave = Execute<ChannelCloseOpenRequest, GenericResponse<int>>(requestCloseOpenChannel, true);
            if (!responseSave.Success)
            {
                returnObject.AddMessage(BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "AccountsParticipationAccountMergeGeneralErrorMessage", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), MessageType.Error, responseSave.Results);
                return returnObject;
            }


            ChannelCloseOpenRequest ChannelCloseOpenRequest2 = new ChannelCloseOpenRequest();
            var requestCloseOpenChannel2 = new ChannelCloseOpenRequest
            {
                UserId = webUser.CustomerPersonId,
                MethodName = "AddUserChannel",

            };
            var responseSave2 = Execute<ChannelCloseOpenRequest, GenericResponse<int>>(requestCloseOpenChannel, true);
            if (!responseSave.Success)
            {
                returnObject.AddMessage(BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "AccountsParticipationAccountMergeGeneralErrorMessage", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), MessageType.Error, responseSave.Results);
                return returnObject;
            }

            return returnObject;
        }

        #endregion
    }
}

view:

@model BOA.Web.InternetBanking.ChannelCloseOpen.Models.IndexModel
@{
    ViewBag.Title = "Index";
}


@using (Html.BeginForm("Index", "ChannelCloseOpen", FormMethod.Post))
{

<div class="col-12">
    <div class="row border border-2 rounded" style="padding-top:14px; padding-bottom:7px;">
        <div class="col-9 fw-bold">@BOA.Web.BusinessHelper.GetMessage("MobilBanking")</div>
        <div class="col-3">
            <div class="row mb-2">
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenMobile, "1", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderOpen", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "1" }, { "checked", "checked" } })
                </div>
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenMobile, "0", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderClose", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "0" },{ "checked", "checked" } })
                </div>
                @Html.HiddenFor(m => m.OpenMobileValue)

            </div>
        </div>
    </div>
    <div style="height:20px" ;></div>
    <div class="row border border-2 rounded" style="padding-top:14px; padding-bottom:7px;">
        <div class="col-9 fw-bold">@BOA.Web.BusinessHelper.GetMessage("InternetBankingText")</div>
        <div class="col-3">
            <div class="row mb-2">
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenInternet, "1", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderOpen", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "1" }, { "checked", "checked" } })
                </div>
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenInternet, "0", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderClose", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "0" }, { "checked", "checked" } })
                </div>
                @Html.HiddenFor(m => m.OpenInternetValue)
            </div>
        </div>
    </div>
    <div style="height:20px" ;></div>
    <div class="row border border-2 rounded" style="padding-top:14px; padding-bottom:7px;">
        <div class="col-9 fw-bold">@BOA.Web.BusinessHelper.GetMessage("API")</div>
        <div class="col-3">
            <div class="row mb-2">
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenApi, "1", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderOpen", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "1" }, { "checked", "checked" } })
                </div>
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenApi, "0", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderClose", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "0" }, { "checked", "checked" } })
                </div>
                @Html.HiddenFor(m => m.OpenApiValue)
            </div>
        </div>
    </div>

    <div style="height:20px" ;></div>
    <div class="row border border-2 rounded" style="padding-top:14px; padding-bottom:7px;">
        <div class="col-9 fw-bold">@BOA.Web.BusinessHelper.GetMessage("PhoneBankingTxt")</div>
        <div class="col-3">
            <div class="row mb-2">
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenTelephone, "1", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderOpen", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "1" }, { "checked", "checked" } })
                </div>
                <div class="col-auto">
                    @Html.BRadioButtonFor(m => m.OpenTelephone, "0", BOA.Messaging.MessagingHelper.GetMessage("InternetBanking", "CreditCardCardInfoUpdateIsMailOrderClose", BOA.Web.Base.Utils.LocalizationHelper.LanguageId), null, null, "", "", null, null, new Dictionary<string, object> { { "id", "0" }, { "checked", "checked" } })
                </div>
                @Html.HiddenFor(m => m.OpenTelephoneValue)
            </div>
        </div>
    </div>

    <div style="height:20px" ;>

    </div>

    @Html.ValidationSummaryArea()
    @Html.NavigationTable(
        "Index",
        BOA.Types.InternetBanking.NavigationTableTypes.PrevType.None,
        BOA.Types.InternetBanking.NavigationTableTypes.NextType.PageToLightBox,
        "")
</div>


}
