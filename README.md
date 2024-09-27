const jwt = require('jsonwebtoken');
const User = require('../models/user.js');
const { dispatchQuery } = require('../path/to/dbConfig'); // Ajuste o caminho

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
        }

        const avatarQuery = `
            SELECT avatarPath 
            FROM UserAvatars 
            WHERE ID_Avatar IN (SELECT ID_Avatar FROM AVATAR_do_COLABORADOR WHERE ID_COLABORADOR = ${user.ID_COLABORADOR})
        `;
        
        const avatarResult = await dispatchQuery(avatarQuery);
        const avatarPath = avatarResult.length > 0 ? avatarResult[0].avatarPath : null;

        res.status(200).json({ auth: true, isFirstAccess, token, userId: user.ID_COLABORADOR, avatarPath });
    } catch (err) {
        console.error('Erro no login:', err);
        res.status(500).json({ auth: false, message: 'Erro interno do servidor' });
    }
}

module.exports = { login };
