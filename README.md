# 🔐 AuthGuard - Sistema Completo de Gerenciamento de Licenças

Clone completo do AuthGuard.org para gerenciamento de keys e licenças premium para scripts.

## ✨ Funcionalidades

### 🔑 Autenticação
- Sistema de login e registro
- JWT tokens com expiração de 30 dias
- Proteção de rotas

### 📦 Gerenciamento de Projetos
- Criar múltiplos projetos/scripts
- Dashboard com estatísticas
- Ver total de keys, keys ativas e validações

### 🎫 Sistema de Keys
- Gerar keys únicas (formato: AGRD-XXXX-XXXX-XXXX)
- Definir duração (dias ou lifetime)
- HWID binding automático na primeira validação
- Status: ativa, expirada, banida
- Notas customizadas por key
- Contador de uso

### 🛠️ Gerenciamento de Keys
- Ver todas as keys de um projeto
- Resetar HWID de uma key
- Deletar keys
- Atualização automática de status (expired)

### 🌐 API de Validação (para seus scripts Lua)
- Endpoint público para validar keys
- Validação de HWID
- Logs de todas as tentativas
- Analytics de uso

## 🚀 Deploy na Vercel

### 1. Configurar MongoDB Atlas

1. Crie uma conta em https://cloud.mongodb.com
2. Crie um cluster grátis
3. Em "Database Access", crie um usuário
4. Em "Network Access", adicione `0.0.0.0/0` (permitir todos os IPs)
5. Copie a connection string

### 2. Deploy na Vercel

1. Faça push deste código para o GitHub
2. Vá em https://vercel.com
3. Import o repositório do GitHub
4. Configure as variáveis de ambiente:

```
MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/authguard
JWT_SECRET=seu_secret_super_seguro_aqui
```

5. Deploy! 🚀

## 📡 Como usar a API no seu script Lua

```lua
local HttpService = game:GetService("HttpService")

local API_URL = "https://seu-projeto.vercel.app/api/validate-key"
local PROJECT_ID = "seu_project_id_aqui" -- Copie do dashboard

-- Obter HWID (exemplo)
local function getHWID()
    return game:GetService("RbxAnalyticsService"):GetClientId()
end

-- Validar key
local function validateKey(key)
    local success, response = pcall(function()
        return HttpService:JSONDecode(
            HttpService:PostAsync(API_URL, HttpService:JSONEncode({
                key = key,
                hwid = getHWID(),
                projectId = PROJECT_ID
            }), Enum.HttpContentType.ApplicationJson)
        )
    end)
    
    if success and response.success then
        print("✅ Key válida! Expira em:", response.expiresAt or "Nunca")
        return true
    else
        warn("❌ Key inválida:", response.error or "Erro desconhecido")
        return false
    end
end

-- Exemplo de uso
local userKey = "AGRD-XXXX-XXXX-XXXX" -- Key que o usuário comprou

if validateKey(userKey) then
    -- Carregar seu script aqui
    print("Script autorizado!")
else
    -- Key inválida
    game.Players.LocalPlayer:Kick("Key inválida ou expirada")
end
```

## 📊 Estrutura do Banco de Dados

### Collection: users
```javascript
{
  _id: ObjectId,
  username: String,
  email: String,
  password: String, // bcrypt hash
  createdAt: Date
}
```

### Collection: projects
```javascript
{
  _id: ObjectId,
  userId: ObjectId,
  name: String,
  description: String,
  createdAt: Date,
  stats: {
    totalKeys: Number,
    activeKeys: Number,
    totalValidations: Number
  }
}
```

### Collection: keys
```javascript
{
  _id: ObjectId,
  projectId: ObjectId,
  userId: ObjectId,
  key: String, // "AGRD-XXXX-XXXX-XXXX"
  hwid: String, // null até primeira validação
  duration: Number, // dias (0 = lifetime)
  status: String, // "active", "expired", "banned"
  createdAt: Date,
  expiresAt: Date, // null se lifetime
  lastUsed: Date,
  usageCount: Number,
  note: String
}
```

### Collection: logs
```javascript
{
  _id: ObjectId,
  projectId: ObjectId,
  keyId: ObjectId,
  key: String,
  hwid: String,
  ip: String,
  action: String, // "validated", "failed", "hwid_mismatch", etc
  reason: String,
  timestamp: Date
}
```

## 🎯 Rotas da API

### Autenticação
- `POST /api/login` - Login
- `POST /api/register` - Registro

### Projetos (requer autenticação)
- `POST /api/create-project` - Criar projeto
- `GET /api/list-projects` - Listar projetos

### Keys (requer autenticação)
- `POST /api/generate-key` - Gerar key
- `GET /api/list-keys?projectId=xxx` - Listar keys
- `POST /api/reset-hwid` - Resetar HWID
- `DELETE /api/delete-key` - Deletar key

### Validação (público)
- `POST /api/validate-key` - Validar key (use no seu script)

## 💡 Dicas

1. **Segurança**: Nunca exponha seu `JWT_SECRET`
2. **MongoDB**: Use senha forte
3. **Keys**: Gere keys com duração adequada (7, 30, 90 dias ou lifetime)
4. **HWID**: Cada key só funciona em um dispositivo após primeira validação
5. **Logs**: Monitore tentativas de validação suspeitas

## 📝 Licença

MIT License - Use livremente!

---

**Desenvolvido com ❤️ para a comunidade de desenvolvedores de scripts Roblox**
