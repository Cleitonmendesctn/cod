const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const User = require('../models/user.js');
const sql = require('mssql/msnodesqlv8'); // Certifique-se de que está importando corretamente

async function login(req, res) {
    const { userEmail, userPassword } = req.body;
    try {
        const user = await User.authenticateUser(userEmail, userPassword);
        if (!user) {
            return res.status(401).json({ auth: false, message: 'Credenciais inválidas' });
        }
        
        const token = jwt.sign({ userId: user.ID_COLABORADOR }, 'secreto', { expiresIn: '1h' });

        let isFirstAccess = false;
        if (userPassword === user.ID_COLABORADOR) {
            isFirstAccess = true;
        } else {
            console.log('Login efetuado');
        }

        // Buscar o avatar do usuário
        const avatarResult = await sql.connect(dbConfig)
            .then(pool => {
                return pool.request()
                    .input('userId', sql.Int, user.ID_COLABORADOR)
                    .query('SELECT avatarPath FROM UserAvatars WHERE ID_Avatar IN (SELECT ID_Avatar FROM AVATAR_do_COLABORADOR WHERE ID_COLABORADOR = @userId)');
            });

        const avatarPath = avatarResult.recordset.length > 0 ? avatarResult.recordset[0].avatarPath : null;

        // Inclua o userId e avatarPath na resposta JSON
        res.status(200).json({ 
            auth: true, 
            isFirstAccess, 
            token, 
            userId: user.ID_COLABORADOR, 
            avatarPath, // Adicione o caminho do avatar aqui
            message: 'Login bem-sucedido' 
        });

    } catch (err) {
        console.error(err);
        res.status(500).json({ auth: false, message: 'Erro interno do servidor' });
    }
}

module.exports = {
    login
};
