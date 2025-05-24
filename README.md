
# Tutorial React com JWT

Esta aplicação é um exemplo básico de um blog onde usuários podem se cadastrar, fazer login, postar mensagens e comentar em mensagens existentes. A autenticação é feita com JWT (JSON Web Token), e o token é armazenado no `localStorage`. A aplicação é protegida por rotas privadas e gerencia o estado de autenticação com Context API.

---

## 📁 Estrutura do Projeto

```
src/
├── index.jsx                 # Ponto de entrada da aplicação
├── App.jsx                   # Configuração de rotas e provedores
├── context/
│   └── AuthContext.jsx       # Contexto de autenticação (login, registro, logout)
└── components/
    ├── Login.jsx             # Tela de login do usuário
    ├── Register.jsx          # Tela de cadastro de usuário
    ├── Messages.jsx          # Exibição e envio de mensagens
    ├── CommentSection.jsx    # Componente para comentários em mensagens
    └── PrivateRoute.jsx      # Proteção de rotas para usuários autenticados
```

---

## 📄 `index.jsx`

Este é o ponto de entrada da aplicação React. Ele renderiza o componente `App` dentro da `div` com id `root`.

---

## 📄 `App.jsx`

Define a estrutura de rotas da aplicação usando React Router. As rotas são:

- `/login`: Acesso à tela de login.
- `/register`: Acesso à tela de cadastro.
- `/`: Página protegida que exibe as mensagens. Usada apenas por usuários logados.

Inclui o `AuthProvider` que envolve toda a aplicação para prover o contexto de autenticação.

---

## 📁 `context/AuthContext.jsx`

Contém o `AuthProvider` e a função `useAuth`.

- Armazena o usuário logado.
- Permite login, registro e logout.
- Armazena e recupera o token JWT do `localStorage`.
- Redireciona com `useNavigate` após ações de login/logout.

---

## 📄 `components/Login.jsx`

Formulário de login de usuário.

- Usa `useAuth()` para chamar a função `login`.
- Redireciona automaticamente após login.
- Link para a tela de registro.

---

## 📄 `components/Register.jsx`

Formulário de registro de novo usuário.

- Usa `useAuth()` para chamar a função `register`.
- Após o registro, o usuário já é autenticado e redirecionado.

---

## 📄 `components/Messages.jsx`

Componente principal da aplicação logada.

- Lista mensagens do backend.
- Permite o envio de novas mensagens.
- Exibe o componente `CommentSection` para cada mensagem.

---

## 📄 `components/CommentSection.jsx`

Permite comentar em uma mensagem específica.

- Lista os comentários da mensagem.
- Permite enviar um novo comentário.
- Utiliza o token JWT no cabeçalho da requisição.

---

## 📄 `components/PrivateRoute.jsx`

Protege rotas que exigem autenticação.

- Verifica se o usuário está presente no contexto.
- Redireciona para `/login` caso contrário.

---

## 🛠️ Tecnologias Utilizadas

- React
- React Router
- Context API
- Axios
- JWT (via headers de autenticação)

---

## ✅ Requisitos

- Um backend que forneça os seguintes endpoints:
  - `POST /api/login`
  - `POST /api/register`
  - `GET /api/messages`
  - `POST /api/messages`
  - `GET /api/messages/:id/comments`
  - `POST /api/messages/:id/comments`

---

## 🔐 Segurança

O token JWT é armazenado no `localStorage` e incluído manualmente no cabeçalho `Authorization` das requisições. Para produção, recomenda-se maior proteção como:

- Expiração automática e renovação de token
- Armazenamento mais seguro (e.g., HTTP-only cookies)

---

## 📌 Observações Finais

Este projeto serve como base educacional. É modular, facilmente expansível, e pode ser integrado com uma API REST moderna em Node.js, Flask, Django, etc.



# 📦 Código Fonte Completo

## `src/index.jsx`

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

## `src/context/AuthContext.jsx`

```jsx
import { createContext, useContext, useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import axios from 'axios';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const navigate = useNavigate();
  const url = "http://localhost:3000";

  const login = async (email, senha) => {
    const response = await axios.post(`${url}/login`, {email, senha });
    localStorage.setItem('token', response.data.token);
    setUser({ email });
    navigate('/');
  };

  const register = async (nome, email, senha) => {
    await axios.post(`${url}/signup`, { nome, email, senha });
    await login(email, senha);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
    navigate('/login');
  };

  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) setUser({});
  }, []);

  return (
    <AuthContext.Provider value={{ user, login, logout, register }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

## `src/components/Login.jsx`

```jsx
import { useState } from 'react';
import { Link } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { login } = useAuth();

  const handleSubmit = (e) => {
    e.preventDefault();
    login(email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
      <button type="submit">Login</button>
      <p>Não tem conta? <Link to="/register">Cadastre-se</Link></p>
    </form>
  );
}
```

## `src/components/Register.jsx`

```jsx
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';

export default function Register() {
  const [nome, setNome] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { register } = useAuth();

  const handleSubmit = (e) => {
    e.preventDefault();
    register(nome, email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="Nome" value={nome} onChange={(e) => setNome(e.target.value)} />
      <input placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
      <button type="submit">Registrar</button>
    </form>
  );
}
```

## `src/components/Messages.jsx`

```jsx
import { useEffect, useState } from 'react';
import axios from 'axios';
import CommentSection from './CommentSection';

export default function Messages() {
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState('');
  const token = localStorage.getItem('token');
  const url = "http://localhost:3000";

  const fetchMessages = async () => {
    const response = await axios.get(`${url}/messages`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    setMessages(response.data);
  };

  const sendMessage = async () => {
    if (newMessage.trim() !== '') {
      await axios.post(`${url}/messages`, { text: newMessage }, {
        headers: { Authorization: `Bearer ${token}` },
      });
      setNewMessage('');
      fetchMessages();
    }
  };

  useEffect(() => {
    fetchMessages();
  }, []);

  return (
    <div>
      <input value={newMessage} onChange={(e) => setNewMessage(e.target.value)} placeholder="Digite sua mensagem" />
      <button onClick={sendMessage}>Enviar</button>
      <ul>
        {messages.map((msg) => (
          <li key={msg.id}>
            {msg.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## `src/components/PrivateRoute.jsx`

```jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function PrivateRoute({ children }) {
  const { user } = useAuth();
  return user ? children : <Navigate to="/login" />;
}
```

## `src/App.jsx`

```jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import Login from './components/Login';
import Register from './components/Register';
import Messages from './components/Messages';
import PrivateRoute from './components/PrivateRoute';

export default function App() {
  return (
    <Router>
      <AuthProvider>
        <Routes>
          <Route path="/login" element={<Login />} />
          <Route path="/register" element={<Register />} />
          <Route path="/" element={<PrivateRoute><Messages /></PrivateRoute>} />
        </Routes>
      </AuthProvider>
    </Router>
  );
}
```
