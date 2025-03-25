dashboards


```
src/
|-- components/
|   |-- Dashboard/
|   |   |-- Dashboard.js         # Componente principal do dashboard
|   |   |-- Dashboard.module.css # Estilos (se usar CSS Modules)
|   |   |-- SummaryCard.js       # Card para métricas rápidas
|   |   |-- SalesChart.js        # Componente do gráfico de vendas
|   |   |-- RecentOrdersTable.js # Tabela de pedidos recentes
|   |-- LoadingSpinner/
|   |   |-- LoadingSpinner.js
|   |   |-- LoadingSpinner.css
|   |-- ErrorMessage/
|   |   |-- ErrorMessage.js
|   |   |-- ErrorMessage.css
|-- hooks/
|   |-- useApiData.js        # Hook customizado para buscar dados (opcional)
|-- services/
|   |-- api.js               # Configuração do Axios e chamadas de API
|-- App.js                   # Componente raiz da aplicação
|-- index.js                 # Ponto de entrada
|-- index.css                # Estilos globais
```

*

```bash
npm install react react-dom axios chart.js react-chartjs-2
# ou
yarn add react react-dom axios chart.js react-chartjs-2
```


```javascript
import axios from 'axios';

// URL base da sua API (substitua pela URL real)
const API_BASE_URL = 'https://api.seudominio.com/v1'; // Exemplo

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
    // Aqui você pode adicionar headers de autenticação (ex: Bearer token)
    // 'Authorization': `Bearer ${getToken()}` // Função para pegar o token
  },
});

// Interceptadores para tratamento de erros global ou refresh de token (opcional)
apiClient.interceptors.response.use(
  response => response,
  error => {
    // Tratamento de erro genérico
    console.error('Erro na chamada da API:', error.response || error.message);
    // Ex: Redirecionar para login se for erro 401 Unauthorized
    // if (error.response && error.response.status === 401) {
    //   window.location = '/login';
    // }
    return Promise.reject(error);
  }
);

// Funções específicas para buscar dados do dashboard
export const fetchSummaryData = () => apiClient.get('/dashboard/summary');
export const fetchSalesData = (period = 'monthly') => apiClient.get(`/dashboard/sales?period=${period}`);
export const fetchRecentOrders = (limit = 10) => apiClient.get(`/orders?limit=${limit}&sort=-createdAt`);

// Exporte o apiClient se precisar usá-lo diretamente em outros lugares
// export default apiClient;
```


`SummaryCard.js`

```javascript''''
import React from 'react';
import styles from './Dashboard.module.css'; // Ou importe seu arquivo CSS

const SummaryCard = ({ title, value, icon, isLoading }) => {
  if (isLoading) {
    return <div className={`${styles.summaryCard} ${styles.loading}`}>Carregando...</div>;
  }

  return (
    <div className={styles.summaryCard}>
      {icon && <div className={styles.cardIcon}>{icon}</div>}
      <div className={styles.cardContent}>
        <h3 className={styles.cardTitle}>{title}</h3>
        <p className={styles.cardValue}>{value}</p>
      </div>
    </div>
  );
};

export default SummaryCard;
```

**`SalesChart.js`** (Usando react-chartjs-2)

```javascript
import React from 'react';
import { Line } from 'react-chartjs-2';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
} from 'chart.js';
import styles from './Dashboard.module.css';

// Registra os componentes necessários do Chart.js
ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
);

const SalesChart = ({ chartData, isLoading }) => {
  if (isLoading) {
    return <div className={`${styles.chartContainer} ${styles.loading}`}>Carregando gráfico...</div>;
  }

  if (!chartData || !chartData.labels || !chartData.values) {
     return <div className={styles.chartContainer}>Dados indisponíveis para o gráfico.</div>;
  }

  const data = {
    labels: chartData.labels, // Ex: ['Jan', 'Fev', 'Mar', ...]
    datasets: [
      {
        label: 'Vendas',
        data: chartData.values, // Ex: [1200, 1900, 3000, ...]
        fill: false,
        borderColor: 'rgb(75, 192, 192)',
        tension: 0.1,
      },
    ],
  };

  const options = {
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
      legend: {
        position: 'top',
      },
      title: {
        display: true,
        text: 'Vendas ao Longo do Tempo',
      },
    },
  };

  return (
    <div className={styles.chartContainer}>
      <Line options={options} data={data} />
    </div>
  );
};

export default SalesChart;
```

**`RecentOrdersTable.js`**

```javascript
import React from 'react';
import styles from './Dashboard.module.css';

const RecentOrdersTable = ({ orders, isLoading }) => {
  if (isLoading) {
    return <div className={`${styles.tableContainer} ${styles.loading}`}>Carregando pedidos...</div>;
  }

   if (!orders || orders.length === 0) {
     return <div className={styles.tableContainer}>Nenhum pedido recente encontrado.</div>;
  }

  return (
    <div className={styles.tableContainer}>
      <h3>Pedidos Recentes</h3>
      <table className={styles.ordersTable}>
        <thead>
          <tr>
            <th>ID Pedido</th>
            <th>Cliente</th>
            <th>Status</th>
            <th>Total</th>
            <th>Data</th>
          </tr>
        </thead>
        <tbody>
          {orders.map((order) => (
            <tr key={order.id}>
              <td>{order.id}</td>
              <td>{order.customerName || 'N/A'}</td>
              <td>
                <span className={`${styles.status} ${styles[order.status?.toLowerCase()]}`}>
                  {order.status || 'N/A'}
                </span>
              </td>
              <td>R$ {order.total?.toFixed(2) || '0.00'}</td>
              <td>{new Date(order.createdAt).toLocaleDateString('pt-BR')}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default RecentOrdersTable;
```

**`Dashboard.js` (Componente Principal)**

```javascript
import React, { useState, useEffect, useCallback } from 'react';
import { fetchSummaryData, fetchSalesData, fetchRecentOrders } from '../../services/api';
import SummaryCard from './SummaryCard';
import SalesChart from './SalesChart';
import RecentOrdersTable from './RecentOrdersTable';
import LoadingSpinner from '../LoadingSpinner/LoadingSpinner'; // Componente de loading genérico
import ErrorMessage from '../ErrorMessage/ErrorMessage';     // Componente de erro genérico
import styles from './Dashboard.module.css';

// Ícones (exemplo simples, use uma biblioteca como react-icons)
const SalesIcon = () => <span>💰</span>;
const UsersIcon = () => <span>👥</span>;
const OrdersIcon = () => <span>📦</span>;

const POLLING_INTERVAL = 30000; // 30 segundos (ajuste conforme necessário)

const Dashboard = () => {
  const [summaryData, setSummaryData] = useState(null);
  const [salesData, setSalesData] = useState(null);
  const [recentOrders, setRecentOrders] = useState();
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  // Usamos useCallback para evitar recriar a função a cada render,
  // o que poderia causar re-execuções desnecessárias do useEffect do polling.
  const loadData = useCallback(async () => {
    // Não mostrar loading em atualizações de polling, apenas na carga inicial
    // setIsLoading(true); // <- Removido daqui
    setError(null);

    try {
      // Busca os dados em paralelo
      const [summaryRes, salesRes, ordersRes] = await Promise.all([
        fetchSummaryData(),
        fetchSalesData('monthly'), // Pode adicionar um seletor de período
        fetchRecentOrders(10),
      ]);

      setSummaryData(summaryRes.data); // Assumindo que a API retorna { data: { totalSales: ..., newUsers: ... } }
      setSalesData(salesRes.data);     // Assumindo { data: { labels: [...], values: [...] } }
      setRecentOrders(ordersRes.data); // Assumindo { data: [...] } (array de pedidos)

    } catch (err) {
      console.error("Falha ao buscar dados do dashboard:", err);
      setError("Não foi possível carregar os dados do dashboard. Tente novamente mais tarde.");
    } finally {
       // Marca como carregado após a primeira busca ou em caso de erro
       if (isLoading) setIsLoading(false);
    }
  }, [isLoading]); // Adiciona isLoading aqui para garantir que setIsLoading(false) seja chamado corretamente na primeira carga

  // Efeito para carregar dados na montagem inicial
  useEffect(() => {
    setIsLoading(true); // Define o loading inicial aqui
    loadData();
  }, [loadData]); // loadData é dependência

  // Efeito para polling (atualização periódica)
  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log('Polling: buscando atualizações de dados...');
      loadData();
    }, POLLING_INTERVAL);

    // Função de limpeza para parar o polling quando o componente desmontar
    return () => clearInterval(intervalId);
  }, [loadData]); // loadData é dependência

  // Renderização condicional
  if (isLoading && !summaryData) { // Mostra loading principal apenas na primeira carga
    return <LoadingSpinner message="Carregando Dashboard..." />;
  }

  if (error) {
    return <ErrorMessage message={error} onRetry={loadData} />;
  }

  // Formatação dos dados do sumário (exemplo)
  const formattedTotalSales = `R$ ${summaryData?.totalSales?.toFixed(2) || '0.00'}`;
  const formattedNewUsers = summaryData?.newUsers || 0;
  const formattedPendingOrders = summaryData?.pendingOrders || 0;

  return (
    <div className={styles.dashboard}>
      <h1>Dashboard Administrativo</h1>

      <div className={styles.summaryGrid}>
        <SummaryCard
            title="Vendas Totais (Mês)"
            value={formattedTotalSales}
            icon={<SalesIcon />}
            isLoading={isLoading && !summaryData} // Loading individual se a carga inicial ainda estiver ocorrendo
        />
        <SummaryCard
            title="Novos Usuários (Mês)"
            value={formattedNewUsers}
            icon={<UsersIcon />}
            isLoading={isLoading && !summaryData}
        />
         <SummaryCard
            title="Pedidos Pendentes"
            value={formattedPendingOrders}
            icon={<OrdersIcon />}
            isLoading={isLoading && !summaryData}
        />
        {/* Adicione mais cards conforme necessário */}
      </div>

      <div className={styles.mainContent}>
        <SalesChart chartData={salesData} isLoading={isLoading && !salesData} />
        <RecentOrdersTable orders={recentOrders} isLoading={isLoading && recentOrders.length === 0} />
      </div>
       {/* Botão para forçar atualização manual (opcional) */}
       <button onClick={loadData} disabled={isLoading}>Atualizar Dados</button>
       {isLoading && <span> Atualizando...</span>}
    </div>
  );
};

export default Dashboard;
```

```css
/* Exemplo de estilos - personalize conforme necessário */
.dashboard {
  padding: 20px;
  font-family: sans-serif;
}

.summaryGrid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.summaryCard {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  transition: transform 0.2s ease-in-out;
}
.summaryCard:hover {
    transform: translateY(-5px);
}

.cardIcon {
  font-size: 2rem;
  margin-right: 15px;
  color: #3498db; /* Cor exemplo */
}

.cardContent {
  flex-grow: 1;
}

.cardTitle {
  margin: 0 0 5px 0;
  font-size: 0.9rem;
  color: #555;
}

.cardValue {
  margin: 0;
  font-size: 1.5rem;
  font-weight: bold;
  color: #333;
}

.mainContent {
  display: grid;
  grid-template-columns: 2fr 1fr; /* Ajuste as colunas */
  gap: 20px;
  margin-bottom: 30px;
}

.chartContainer {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  height: 400px; /* Defina uma altura */
  position: relative; /* Para posicionar loading/error */
}

.tableContainer {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow-x: auto; /* Para tabelas largas em telas pequenas */
}

.ordersTable {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
}

.ordersTable th,
.ordersTable td {
  border-bottom: 1px solid #eee;
  padding: 10px 8px;
  text-align: left;
}

.ordersTable th {
  font-weight: bold;
  background-color: #f8f8f8;
}

.status {
    padding: 3px 8px;
    border-radius: 12px;
    font-size: 0.8rem;
    font-weight: bold;
    color: #fff;
    text-transform: capitalize;
}

.status.processing { background-color: #3498db; }
.status.completed { background-color: #2ecc71; }
.status.shipped { background-color: #f39c12; }
.status.cancelled { background-color: #e74c3c; }
.status.pending { background-color: #95a5a6; }


/* Estilos de Loading/Error */
.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100px; /* Altura mínima para visibilidade */
  color: #777;
}

/* Adicione estilos para LoadingSpinner.css e ErrorMessage.css */

```

```javascript
import React from 'react';
import Dashboard from './components/Dashboard/Dashboard';
import './index.css'; // Estilos globais

function App() {
  // Aqui você poderia ter Lógica de Roteamento, Layout Principal, etc.
  return (
    <div className="App">
      {/* Ex: <Navbar /> */}
      <Dashboard />
      {/* Ex: <Footer /> */}
    </div>
  );
}

export default App;
```
