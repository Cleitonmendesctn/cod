const removeNotificationFromList = (idNotification) => {
    setNotifications((prev) => prev.filter(not => not.ID_NOTIFICACAO !== idNotification));
};


const handleExcludeNotification = async () => { 
    const token = sessionStorage.getItem('token'); 
    const response = await fetchDeleteNotification({ 
        token, 
        notificationId: notification?.ID_NOTIFICACAO, 
        serverIP 
    });

    if (!response.ok) {
        alert('Erro ao excluir notificação');
    } else {
        handleExclusion(notification?.ID_NOTIFICACAO); // Atualiza a lista no componente pai
    }
};
