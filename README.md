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
        
