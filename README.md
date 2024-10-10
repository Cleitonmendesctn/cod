import { useEffect, useState, useRef } from 'react';
import './notification.css';
import notification_foto from '../../../public/img/svgs/notificacao.svg';
import NotificationItem from './notificationItem/NotificationItem';
import React from 'react';

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
                <img src={notification_foto} />
                {notification?.length > 0 ? 
                    <div className='notification_number'>{notification.length}</div> 
                : ''}
            </div>
            <div className='notifications_dropdown' style={isMenuOpened ? {} : { display: 'none' }}>
                <div className='arrow'></div>
                {notification?.length > 0 ? 
                    notification.map((item, index) => (
                        <React.Fragment key={`notification_${item?.TEXTO}_${index}`}>
                            <NotificationItem 
                                notification={item} 
                                handleExclusion={handleExclusion} 
                                serverIP={serverIP} 
                            />
                            {index !== notification.length - 1 ? <div className='division'></div> : ''}
                        </React.Fragment>
                    )) 
                : 'Sem notificações pendentes'}
            </div>
        </div>
    );
}
