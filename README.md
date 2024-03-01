  public GenericResponse<List<ChannelCloseOpenContract>> GetChannelCloseOpen(int userId)
        {
            SqlCommand command;
            GenericResponse<List<ChannelCloseOpenContract>> returnObject;
            GenericResponse<SqlDataReader> sp;
            returnObject = this.InitializeGenericResponse<List<ChannelCloseOpenContract>>("BOA.Business.InternetBanking.UserSettings.ChannelCloseOpen.GetChannelCloseOpen");
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


           public class ChannelCloseOpenContract : ContractBase
    {


        public int ChannelId { get; set; }
        public string ChannelName { get; set; }
        public int UserChannelId { get; set; }

        public Int32 Id
        {
            get;
            set;
        }

        public Int32 UserId
        {
            get;
            set;
        }


        public Int16 OtpStateId
        {
            get;
            set;
        }

        public Int16 MobileSignStateId
        {
            get;
            set;
        }

        public Int16 SoftOtpStateId
        {
            get;
            set;
        }

        public Int16? LastLoginType
        {
            get;
            set;
        }

           public class ChannelCloseOpenRequest : RequestBase
    {
        public Int32 UserId { get; set; }
        public Byte ChannelId { get; set; }
        public int SoftOtpStateId { get; set; }

        public int UserChannelId { get; set; }
        public string ChannelName { get; set; }

        public ChannelCloseOpenContract ChannelContract { get; set; }


    }

      public GenericResponse<List<ChannelCloseOpenContract>> SelectChannelCloseOpen(ChannelCloseOpenRequest request, ObjectHelper objectHelper)
        {
            GenericResponse<List<ChannelCloseOpenContract>> returnObject = objectHelper.InitializeGenericResponse<List<ChannelCloseOpenContract>>("BOA.Orchestration.InternetBanking.UserSettings.ChannelCloseOpen.SelectChannelCloseOpen");
            BOA.Business.InternetBanking.UserSettings.ChannelCloseOpen bo = new  Business.InternetBanking.UserSettings.ChannelCloseOpen(objectHelper.Context);
            GenericResponse<List<BOA.Types.InternetBanking.UserSettings.ChannelCloseOpenContract>> responseChannel = bo.GetChannelCloseOpen(request.UserId);
            if (!responseChannel.Success)
            {
                returnObject.Results.AddRange(responseChannel.Results);
                return returnObject;
            }
            returnObject.Value = responseChannel.Value;
            return returnObject;
        }

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
