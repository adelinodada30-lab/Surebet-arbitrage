// Elementos do DOM
const eventsContainer = document.getElementById('events-container');
const profitSlider = document.getElementById('profit');
const profitValue = document.getElementById('profit-value');
const stakeInput = document.getElementById('stake');
const sportSelect = document.getElementById('sport');
const apiKeyInput = document.getElementById('api-key');

// Configurações
let stakeValue = 100;
let minProfit = 2;

// Inicialização
document.addEventListener('DOMContentLoaded', function() {
    profitSlider.addEventListener('input', updateProfitValue);
    stakeInput.addEventListener('input', updateStakeValue);
});

function updateProfitValue() {
    minProfit = parseInt(profitSlider.value);
    profitValue.textContent = `${minProfit}%`;
}

function updateStakeValue() {
    stakeValue = parseFloat(stakeInput.value);
}

// Testar chave API
async function testApiKey() {
    const apiKey = apiKeyInput.value.trim();
    if (!apiKey) {
        showError('Por favor, insira uma chave API');
        return;
    }
    
    showLoading('Testando conexão com a API...');
    
    try {
        const response = await fetch(`https://api.allorigins.win/get?url=${encodeURIComponent(`https://api.the-odds-api.com/v4/sports?apiKey=${apiKey}`)}`);
        
        if (!response.ok) throw new Error('Erro na API');
        
        const data = await response.json();
        const sports = JSON.parse(data.contents);
        
        showSuccess(`Conexão bem-sucedida! ${sports.length} esportes disponíveis`);
        
    } catch (error) {
        showError('Erro na conexão. Verifique sua chave API.');
    }
}

// Carregar eventos
async function loadEvents() {
    const apiKey = apiKeyInput.value.trim();
    const sport = sportSelect.value;
    
    if (!apiKey) {
        showError('Por favor, insira uma chave API válida');
        return;
    }
    
    showLoading('Buscando odds em tempo real...');
    
    try {
        const apiUrl = `https://api.the-odds-api.com/v4/sports/${sport}/odds/?apiKey=${apiKey}&regions=eu&markets=h2h&oddsFormat=decimal`;
        const response = await fetch(`https://api.allorigins.win/get?url=${encodeURIComponent(apiUrl)}`);
        
        if (!response.ok) throw new Error('Erro ao carregar dados');
        
        const data = await response.json();
        const events = JSON.parse(data.contents);
        
        if (events.length === 0) {
            showError('Nenhum evento encontrado. Tente outro esporte.');
            return;
        }
        
        displayEvents(events);
        
    } catch (error) {
        console.error('Erro:', error);
        showError('Erro ao carregar dados. Usando dados de exemplo...');
        setTimeout(() => useSampleData(), 2000);
    }
}

// Exibir eventos
function displayEvents(events) {
    let html = '';
    let opportunityCount = 0;
    
    events.forEach(event => {
        const arbitrage = calculateArbitrage(event.bookmakers, stakeValue);
        
        if (arbitrage.profit >= minProfit) {
            opportunityCount++;
            html += createEventCard(event, arbitrage);
        }
    });
    
    if (opportunityCount === 0) {
        showError('Nenhuma oportunidade de arbitragem encontrada');
    } else {
        eventsContainer.innerHTML = html;
    }
}

// Criar card de evento
function createEventCard(event, arbitrage) {
    return `
        <div class="event-card">
            <div class="event-header">
                <div><i class="fas fa-${getSportIcon(event.sport_title)}"></i> ${event.sport_title}</div>
                <div>${new Date(event.commence_time).toLocaleTimeString()}</div>
            </div>
            <div class="event-body">
                <div class="event-teams">${event.home_team} vs ${event.away_team}</div>
                
                <div class="arbitrage-opportunity">
                    <i class="fas fa-money-bill-wave"></i> Oportunidade de Arbitragem
                    <span class="profit-badge">${arbitrage.profit}%</span>
                </div>
                
                <div class="stake-calculation">
                    <div><strong>Investimento Total:</strong> R$ ${arbitrage.totalStake}</div>
                    <div><strong>Retorno Garantido:</strong> R$ ${arbitrage.expectedReturn}</div>
                    <div><strong>Lucro:</strong> R$ ${(arbitrage.expectedReturn - arbitrage.totalStake).toFixed(2)} (${arbitrage.profit}%)</div>
                </div>
                
                <div class="bet-links">
                    <a href="#" class="bet-link" onclick="alert('Redirecionando para Bet365...')">Bet365</a>
                    <a href="#" class="bet-link" onclick="alert('Redirecionando para Betano...')">Betano</a>
                    <a href="#" class="bet-link" onclick="alert('Redirecionando para William Hill...')">William Hill</a>
                </div>
            </div>
        </div>
    `;
}

// Calcular arbitragem
function calculateArbitrage(bookmakers, stake) {
    let bestHome = 0, bestAway = 0, bestDraw = 0;
    
    bookmakers.forEach(bookmaker => {
        if (bookmaker.markets?.[0]?.outcomes) {
            const outcomes = bookmaker.markets[0].outcomes;
            if (outcomes[0]?.price > bestHome) bestHome = outcomes[0].price;
            if (outcomes[1]?.price > bestAway) bestAway = outcomes[1].price;
            if (outcomes[2]?.price > bestDraw) bestDraw = outcomes[2].price;
        }
    });
    
    if (bestDraw > 0) {
        return calculateThreeWayArbitrage(bestHome, bestDraw, bestAway, stake);
    } else {
        return calculateTwoWayArbitrage(bestHome, bestAway, stake);
    }
}

// Cálculos específicos
function calculateThreeWayArbitrage(home, draw, away, stake) {
    const total = (1/home) + (1/draw) + (1/away);
    const profitPercentage = (1 - total) * 100;
    
    if (total < 1 && profitPercentage >= minProfit) {
        const homeStake = (stake / home) / total;
        const drawStake = (stake / draw) / total;
        const awayStake = (stake / away) / total;
        
        return {
            arbitrage: true,
            profit: profitPercentage.toFixed(2),
            totalStake: (homeStake + drawStake + awayStake).toFixed(2),
            expectedReturn: (stake / total).toFixed(2)
        };
    }
    
    return { arbitrage: false, profit: 0, totalStake: 0, expectedReturn: 0 };
}

function calculateTwoWayArbitrage(home, away, stake) {
    const total = (1/home) + (1/away);
    const profitPercentage = (1 - total) * 100;
    
    if (total < 1 && profitPercentage >= minProfit) {
        const homeStake = (stake / home) / total;
        const awayStake = (stake / away) / total;
        
        return {
            arbitrage: true,
            profit: profitPercentage.toFixed(2),
            totalStake: (homeStake + awayStake).toFixed(2),
            expectedReturn: (stake / total).toFixed(2)
        };
    }
    
    return { arbitrage: false, profit: 0, totalStake: 0, expectedReturn: 0 };
}

// Utilitários
function showLoading(message) {
    eventsContainer.innerHTML = `
        <div class="loading">
            <i class="fas fa-spinner fa-spin fa-3x"></i>
            <p>${message}</p>
        </div>
    `;
}

function showError(message) {
    eventsContainer.innerHTML = `
        <div class="error">
            <i class="fas fa-exclamation-triangle"></i> ${message}
        </div>
    `;
}

function showSuccess(message) {
    eventsContainer.innerHTML = `
        <div class="arbitrage-opportunity">
            <i class="fas fa-check-circle"></i> ${message}
        </div>
    `;
}

function getSportIcon(sport) {
    const icons = {
        'soccer': 'futbol',
        'basketball': 'basketball-ball',
        'tennis': 'table-tennis',
        'icehockey': 'hockey-puck'
    };
    return icons[sport] || 'running';
}

// Fallback para dados de exemplo
function useSampleData() {
    const sampleEvents = [{
        sport_title: "Futebol",
        home_team: "Flamengo",
        away_team: "Palmeiras",
        commence_time: new Date(),
        bookmakers: [{
            key: "bet365",
            markets: [{
                outcomes: [
                    { name: "Flamengo", price: 2.10 },
                    { name: "Palmeiras", price: 3.80 },
                    { name: "Draw", price: 3.25 }
                ]
            }]
        }, {
            key: "betano",
            markets: [{
                outcomes: [
                    { name: "Flamengo", price: 2.30 },
                    { name: "Palmeiras", price: 3.20 },
                    { name: "Draw", price: 3.10 }
                ]
            }]
        }]
    }];
    
    displayEvents(sampleEvents);
}