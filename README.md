const fetchAvatar = async (req, res) => {
    try {
        const userId = req.userId;

        // Primeiro, busque o ID_Avatar na tabela AVATAR_do_COLABORADOR
        const avatarData = await avatarModel.getAvatar(userId);
        
        if (avatarData && avatarData.length > 0) {
            const avatarId = avatarData[0].ID_Avatar;

            // Agora busque o caminho da imagem usando o avatarId
            const avatarDetails = await avatarModel.getAvatarDetails(avatarId);
            if (avatarDetails && avatarDetails.length > 0) {
                res.status(200).json({ avatarPath: avatarDetails[0].avatarPath });
            } else {
                res.status(404).json({ message: 'Detalhes do avatar não encontrados' });
            }
        } else {
            res.status(404).json({ message: 'Avatar não encontrado' });
        }
    } catch (err) {
        console.log(err);
        res.status(500).json({ message: 'Erro ao buscar avatar' });
    }
};

\\MODEL: async function getAvatarDetails(avatarId) {
    const sql = `SELECT avatarPath FROM UserAvatars WHERE ID_Avatar = '${avatarId}'`;
    const results = await sqlServer.dispatchQuery(sql);
    return results;
}


\\AVATAR FRONT:
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
            if (data && data.avatarPath) {
                setAvatar(data.avatarPath); // Armazene o caminho da imagem
            } else {
                console.error('Caminho do avatar não encontrado na resposta:', data);
            }
        })
        .catch(error => console.error('Erro ao buscar o avatar:', error));
    } else {
        console.error('User ID não encontrado no sessionStorage');
    }
}, [serverIP, token]);

\\ BOX PERFIL: import usuario from '/img/svgs/avatarmasculino.png';
import BarraProgresso from '../Progresso/BarraProgresso';
import coin from '/img/svgs/Dolar_Dinero_Moneda_1Light.svg';
import { Link } from 'react-router-dom';
import { useState, useEffect } from 'react';
import '../BoxPerfil/boxperfil.css';

function BoxPerfil({ serverIP, avatar }) {
  const [nivel, setNivel] = useState('');
  const [xp, setXp] = useState('');
  const [moedas, setMoedas] = useState('');
  const [userName, setUsername] = useState('');
  const [currentAvatar, setCurrentAvatar] = useState(usuario); // Avatar padrão

  const token = sessionStorage.getItem('token');

  useEffect(() => {
    async function fetchData() {
      try {
        const response = await fetch(`${serverIP}/getUserData`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'x-access-token': token
          }
        });
        const data = await response.json();
        setUsername(data.NOME);
        sessionStorage.setItem('username', data.NOME);
        setMoedas(data.MOEDAS);
        sessionStorage.setItem('usermoedas', data.MOEDAS);
        setNivel(data.NIVEL);
        sessionStorage.setItem('usernivel', data.NIVEL);
        setXp(data.XP);
        sessionStorage.setItem('userxp', data.XP);
      } catch (error) {
        console.log('Erro ao buscar os dados:', error);
      }
    }

    fetchData();
  }, [serverIP]);

  useEffect(() => {
    // Atualiza o avatar com base no que vem do props
    if (avatar) {
      setCurrentAvatar(avatar);
    } else {
      setCurrentAvatar(usuario); // Se não houver avatar, usa o padrão
    }
  }, [avatar]);

  return (
    <div>
      <Link to="/Perfil" style={{ textDecoration: 'none' }}>
        <header className="header-perfil">
          <img className="icon-usuario" src={currentAvatar} alt="usuario" />
          <div className="info">
            <div className="nome-e-nivel">
              <h2 className="subinfo typing-effect">{userName}</h2>
              <h4 className="subinfo">Nível {nivel}</h4>
            </div>
            <div className='subinfo-progresso'>
              <h4 className="subinfo">EXP </h4>
              <BarraProgresso xp={xp} />
            </div>
            <div className="coin-valor">
              <img className='coin' src={coin} alt="Ícone de Moedas" />
              {moedas}
            </div>
          </div>
        </header>
      </Link>
    </div>
  );
}

export default BoxPerfil;

