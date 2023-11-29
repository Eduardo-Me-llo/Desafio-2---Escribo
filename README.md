# Desafio-2---Escribo

document.addEventListener('DOMContentLoaded', () => {
  const signupForm = document.getElementById('signupForm');
  const signinForm = document.getElementById('signinForm');
  const getUserInfoBtn = document.getElementById('getUserInfo');
  const userInfo = document.getElementById('userInfo');

  signupForm.addEventListener('submit', async (event) => {
    event.preventDefault();
    const email = document.getElementById('signupEmail').value;
    const password = document.getElementById('signupPassword').value;

    try {
      const response = await fetch('/signup', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, password })
      });

      const data = await response.json();
      alert(data.mensagem);
    } catch (error) {
      console.error('Erro ao cadastrar usuário:', error);
    }
  });

  signinForm.addEventListener('submit', async (event) => {
    event.preventDefault();
    const email = document.getElementById('signinEmail').value;
    const password = document.getElementById('signinPassword').value;

    try {
      const response = await fetch('/signin', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, password })
      });

      const data = await response.json();
      alert(data.token ? 'Usuário autenticado com sucesso' : data.mensagem);
    } catch (error) {
      console.error('Erro ao autenticar usuário:', error);
    }
  });

  getUserInfoBtn.addEventListener('click', async () => {
    try {
      const token = prompt('Insira o token de autenticação');
      const response = await fetch('/user', {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${token}`
        }
      });

      const data = await response.json();
      userInfo.textContent = data.id ? `ID do Usuário: ${data.id}, E-mail: ${data.email}` : data.mensagem;
    } catch (error) {
      console.error('Erro ao buscar informações do usuário:', error);
    }
  });
});
// Simulando armazenamento temporário dos usuários em um array
let users = [];

app.post('/signup', async (req, res) => {
  try {
    const { email, password } = req.body;

    // Verificar se o e-mail já está cadastrado
    const existingUser = users.find(user => user.email === email);
    if (existingUser) {
      return res.status(400).json({ mensagem: 'E-mail já existente' });
    }

    // Criptografar a senha (simulação, utilize uma solução mais segura em produção)
    const hashedPassword = await bcrypt.hash(password, 10);

    const newUser = {
      id: users.length + 1,
      email,
      password: hashedPassword,
    };

    users.push(newUser);
    res.status(201).json({ mensagem: 'Usuário cadastrado com sucesso' });
  } catch (error) {
    res.status(500).json({ mensagem: 'Erro ao cadastrar usuário' });
  }
});

app.post('/signin', async (req, res) => {
  try {
    const { email, password } = req.body;

    const user = users.find(user => user.email === email);
    if (!user) {
      return res.status(401).json({ mensagem: 'Usuário e/ou senha inválidos' });
    }

    const passwordValid = await bcrypt.compare(password, user.password);
    if (!passwordValid) {
      return res.status(401).json({ mensagem: 'Usuário e/ou senha inválidos' });
    }

    const token = jwt.sign({ email: user.email }, 'secret', { expiresIn: '30m' });

    res.json({ token });
  } catch (error) {
    res.status(500).json({ mensagem: 'Erro ao autenticar usuário' });
  }
});
