CREATE PROCEDURE dbo.SetChannelState
    @UserId INT,
    @ChannelId BYTE,
    @IsOpen BIT
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @UserChannelId INT;

    -- Kullanıcının belirli bir kanalın durumunu al
    SELECT @UserChannelId = UserChannelId
    FROM YourUserChannelTable
    WHERE UserId = @UserId AND ChannelId = @ChannelId;

    -- Eğer kanal kaydı yoksa oluştur
    IF @UserChannelId IS NULL
    BEGIN
        INSERT INTO YourUserChannelTable (UserId, ChannelId, IsOpen)
        VALUES (@UserId, @ChannelId, @IsOpen);
    END
    ELSE
    BEGIN
        -- Kanal kaydı varsa güncelle
        UPDATE YourUserChannelTable
        SET IsOpen = @IsOpen
        WHERE UserChannelId = @UserChannelId;
    END
END;
