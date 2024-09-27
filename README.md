    const fetchAvatar = async (userId) => {
        const token = sessionStorage.getItem("token");
        try {
            const response = await fetch(`${serverIP}/avatar/get-avatar?userId=${userId}`, {
                headers: {
                    'x-access-token': token
                }
            });
            if (!response.ok) {
                throw new Error('Erro ao buscar o avatar');
            }
            const data = await response.json();
            if (data && data.avatarPath) {
                setAvatar(data.avatarPath); // Armazena o caminho do avatar
            } else {
                console.error('Caminho do avatar não encontrado na resposta:', data);
            }
        } catch (error) {
            console.error('Erro ao buscar o avatar:', error);
        }
    };
