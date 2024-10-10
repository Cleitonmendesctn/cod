const handleExcludeNotification = async () => {
    const token = sessionStorage.getItem('token');
    const response = await fetchDeleteNotification({ token, notification: notification?.ID_NOTIFICACAO, serverIP });

    if (!response.ok) {
        alert('Erro ao excluir notificação');
    } else {
        handleExclusion(notification?.ID_NOTIFICACAO); // Chama a função passada como prop
    }
};
