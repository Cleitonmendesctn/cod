import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { FaTrash } from "react-icons/fa6";
import fetchReadNotification from "../../../services/notifications/fetchReadNotification";
import fetchDeleteNotification from "../../../services/notifications/fetchDeleteNotification";

export default function NotificationItem({ serverIP, notification, handleExclusion }) {
    const navigate = useNavigate();
    const [isRead, setIsRead] = useState(notification?.STATUS_LEITURA);

    const handleOnClick = async () => {
        if (!notification?.STATUS_LEITURA) {
            const token = sessionStorage.getItem('token');
            const response = await fetchReadNotification(token, notification?.ID_NOTIFICACAO);
            if (!response.ok) alert("Erro ao ler notificação");
            else setIsRead(true);
        }
        if (notification.REFERENCIA.path) navigate('/' + notification.REFERENCIA.path);
    };

    const handleExcludeNotification = async () => {
        const token = sessionStorage.getItem('token');
        console.log('Excluindo notificação:', notification?.ID_NOTIFICACAO);
        console.log('Server IP:', serverIP); // Verifique se está definido

        const response = await fetchDeleteNotification({
            token,
            notificationId: notification?.ID_NOTIFICACAO,
            serverIP
        });

        if (!response.ok) {
            alert('Erro ao excluir notificação');
        } else {
            handleExclusion(notification?.ID_NOTIFICACAO); // Chama a função passada como prop
        }
    };

    return (
        <div className="notification_item" style={isRead ? { color: 'var(--vivo-lightpurple50)' } : { color: 'var(--vivo-green)', fontWeight: '600' }}>
            <div className="text" onClick={handleOnClick}>
                {notification?.TEXTO}
            </div>
            <div className="actions">
                <FaTrash className="trash" onClick={handleExcludeNotification} />
            </div>
        </div>
    );
}


import React, { useEffect, useState, useRef } from 'react';
import './notification.css';
import notification_foto from '../../../public/img/svgs/notificacao.svg';
import NotificationItem from './notificationItem/NotificationItem';

export default function Notifications({ notification, handleExclusion, serverIP }) {
    const divRef = useRef(null);
    const [isMenuOpened, setIsMenuOpened] = useState(false);

    const handleClickOutside = (e) => {
        if (divRef.current && !divRef.current.contains(e.target)) {
            setIsMenuOpened(false);
        }
    };

    useEffect(() => {
        document.addEventListener('mousedown', handleClickOutside);
        return () => {
            document.removeEventListener('mousedown', handleClickOutside);
        };
    }, []);

    return (
        <div className='notifications_menu' ref={divRef}>
            <div className='notification_icon' onClick={() => setIsMenuOpened(prev => !prev)}>
                <img src={notification_foto} alt="Notificações" />
                {notification?.length > 0 && (
                    <div className='notification_number'>{notification.length}</div>
                )}
            </div>
            <div className='notifications_dropdown' style={isMenuOpened ? {} : { display: 'none' }}>
                <div className='arrow'></div>
                {notification?.length > 0 ? (
                    notification.map((item, index) => (
                        <React.Fragment key={`notification_${item?.TEXTO}_${index}`}>
                            <NotificationItem
                                notification={item}
                                handleExclusion={handleExclusion}
                                serverIP={serverIP} // Passando serverIP aqui
                            />
                            {index !== notification.length - 1 && <div className='division'></div>}
                        </React.Fragment>
                    ))
                ) : (
                    'Sem notificações pendentes'
                )}
            </div>
        </div>
    );
}
