using BOA.Common.Types;
using BOA.Types.InternetBanking;
using BOA.Types.InternetBanking.UserSettings;
using BOA.Web.Base;
using BOA.Web.Base.Types;
using BOA.Web.Base.Utils;
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

            TransactionContext.CurrentTransactionCode = TransactionCode.CHANNEL_CLOSE_OPEN;
            var indexModel = GetModel<IndexModel>(IndexView.Name);


            if (indexModel == null)
            {
                indexModel = new IndexModel();
            }


      


            WebUserContract webUser = WebContext.UserDataDictionary[BOA.Web.InternetBanking.Types.SessionKeys.WebUser] as WebUserContract;

            ChannelCloseOpenRequest ChannelRequest = new ChannelCloseOpenRequest();
            var requestGetChannel = new ChannelCloseOpenRequest
            {
                UserId = webUser.CustomerPersonId,
                MethodName = "SelectChannelCloseOpen",
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


conroller2


  public static GenericResponse<UserChannelContract> GetUserChannel(int userId, byte channelId)
        {
            UserChannelRequest request = new UserChannelRequest {
                MethodName = "GetUserChannel",
                MainAccountNumber = SessionManager.WebContext.UserContract.CustomerId,
                UserId = userId,
                ChannelId = channelId
            };
            var response = BExecuter.Execute<UserChannelRequest, GenericResponse<UserChannelContract>>(request);
            if (!response.Success)
            {
                BOAWebHelper.Log("BOA.Web.InternetBanking.BusinessHelper.GetUserChannel error!", response.Results);
            }
            return response;
        }
