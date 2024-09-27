import { useState } from 'react';
import Swal from 'sweetalert2';
import EmailInput from '../EmailInput/EmailInput';
import PasswordInput from '../PasswordInput/passwordInput';
import Logo from '/img/svgs/logoprogressao.png';
import { useAvatar } from '../../Context/AvatarContext'; // Ajuste o caminho conforme necessário

function LoginForm({ serverIP }) {
    const [userEmail, setUserEmail] = useState('');
    const [userPassword, setUserPassword] = useState('');
    const [loginError, setLoginError] = useState(false);
    const { setAvatar } = useAvatar(); // Hook para acessar o contexto de avatar

    const handleSubmit = async (event) => {
        event.preventDefault();

        try {
            const response = await fetch(`${serverIP}/login`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ userEmail: userEmail, userPassword: userPassword })
            });

            const data = await response.json();
            console.log('Resposta da API:', data); 

            if (response.ok) {
                sessionStorage.setItem('token', data.token);
                sessionStorage.setItem('userId', data.userId);

                // Defina o avatar logo após o login
                if (data.avatarPath) {
                    setAvatar(data.avatarPath);
                    sessionStorage.setItem('avatar', data.avatarPath); // Opcional
                }

                if (data.isFirstAccess) {
                    Swal.fire({
                        title: 'Aviso!',
                        text: 'Por favor, altere sua senha após o primeiro acesso.',
                        icon: 'warning',
                        confirmButtonText: 'OK',
                    }).then(() => {
                        window.location.href = '/update';
                    });
                } else {
                    window.location.href = '/missoes';
                }
            } else {
                setLoginError(true);
                console.log('Falha no login:', data.message);
            }
        } catch (error) {
            console.error('Erro ao fazer login:', error);
            alert("Ocorreu um erro ao fazer login. Por favor, tente novamente.");
        }
    };

    return (
        <div className="login-container">
            <form className='login-form' onSubmit={handleSubmit}>
                <img src={Logo} className="vivo-icon" alt="vivo icon" />
                <EmailInput
                    value={userEmail}
                    onChange={setUserEmail}
                />
                <PasswordInput
                    state={userPassword}
                    onChange={setUserPassword}
                    placeholder={"Insira sua senha"}
                    showRegexError={false}
                />
                <button type='submit'>Acessar</button>
            </form>
            {loginError && <p className="login-error">Email ou senha incorretos</p>}
        </div>
    );
}

export default LoginForm;
