  const handleExclusion = (idNotification) => {
    setNotifications((prevNotifications) => 
      prevNotifications.filter(notification => notification.ID_NOTIFICACAO !== idNotification)
    );
  };
