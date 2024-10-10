export default async function fetchDeleteNotification({token, notificationId, serverIP}) {
    try {
        const response = await fetch(`${serverIP}/deleteNotification/${notificationId}`, {
            method: 'DELETE',
            headers: { 'Content-Type': 'application/json', 'x-access-token': token }
        });
        return response;
    } catch (error) {
        console.error(error);
        return null;
    }
}
