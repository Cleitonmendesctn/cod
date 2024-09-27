import React, { useState, useEffect } from 'react';
import BoxPerfil from "../BoxPerfil/BoxPerfil";
import Navmenu from '../../Navbar/Navmenu';
import AvatarSelector from './AvatarSelector';
import './Avatar.css';
import Swal from 'sweetalert2';
import BackArrow from "/img/svgs/voltar.svg";
import LogoutButton from "../../userSessions/Logout/LogoutButton";
import { useAvatar } from '../../Context/AvatarContext';

export default function Avatar({ serverIP }) {
    const { avatar, setAvatar } = useAvatar();
    const token = sessionStorage.getItem("token");
    const userId = sessionStorage.getItem('userId');
    const [selectedAvatar, setSelectedAvatar] = useState(null);

    if (!token) {
        window.location.href = "/";
    }

    useEffect(() => {
        if (token) {
            fetch(`${serverIP}/avatar/get-avatar?userId=${userId}`, {
                headers: {
                    'x-access-token': token
                }
            })
            .then(response => {
                if (!response.ok) {
                    throw new Error('Erro ao buscar o avatar');
                }
                return response.json();
            })
            .then(data => {
                if (data && data.avatarId) {
                    // Atualize para usar avatarId em vez de avatarPath
                    setAvatar(data.avatarId); // Armazene o ID do avatar
                } else {
                    console.error('ID do avatar não encontrado na resposta:', data);
                }
            })
            .catch(error => console.error('Erro ao buscar o avatar:', error));
        } else {
            console.error('User ID não encontrado no sessionStorage');
        }
    }, [serverIP, token, setAvatar, userId]); // Inclua userId como dependência

    const handleAvatarSelect = (avatar) => {
        setSelectedAvatar(avatar);
    };

    const handleSaveAvatar = async () => {
        if (selectedAvatar && userId && token) {
            try {
                const response = await fetch(`${serverIP}/avatar/set-avatar`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'x-access-token': token
                    },
                    body: JSON.stringify({ userId: userId, avatarId: selectedAvatar })
                });

                if (response.ok) {
                    const data = await response.json();
                    setAvatar(selectedAvatar); // Atualize o avatar no contexto
                    sessionStorage.setItem('avatar', selectedAvatar);
                    Swal.fire({
                        icon: 'success',
                        title: 'Alterado!',
                        text: 'Avatar atualizado com sucesso!',
                    });
                } else {
                    const errorData = await response.json();
                    console.log('erro na resposta:', errorData);
                    Swal.fire({
                        icon: 'error',
                        title: 'Erro!',
                        text: `Erro ao atualizar o avatar: ${response.statusText}`,
                    });
                }
            } catch (error) {
                Swal.fire({
                    icon: 'error',
                    title: 'Erro!',
                    text: `Erro ao atualizar o avatar: ${error.message}`,
                });
            }
        } else {
            Swal.fire({
                icon: 'error',
                title: 'Erro!',
                text: 'User ID ou token não encontrado. Por favor, faça login novamente.',
            });
        }
    };

    return (
        <div className="todocontainer">
            <BoxPerfil serverIP={serverIP} avatar={avatar} />
            <Navmenu />
            <div className='header-avatar'>
                <div id="sair-app">
                    <a href="/perfil">
                        <img className="btn-backPage" src={BackArrow} alt="Voltar" />
                    </a>
                    <h2 id="titulopagina">Selecione seu Avatar</h2>
                    <LogoutButton />
                </div>
                <div className='avatares-options'>
                    <AvatarSelector onSelect={handleAvatarSelect} />
                    {selectedAvatar && <p>Avatar Selecionado: {selectedAvatar.split('/').pop().replace('.png', '')}</p>}
                </div>
                <button className='alterar-avatar' onClick={handleSaveAvatar}>
                    Alterar Avatar
                </button>
            </div>
        </div>
    );
}
