# 🎈 KembPro8

// Projeto: MarketMind - Gerador Inteligente de Conteúdo de Marketing

import { initializeApp } from 'firebase/app';
import { getFirestore, collection, addDoc } from 'firebase/firestore';
import { configuration, OpenAIApi } from 'openai';

// Configuração do Firebase
const firebaseConfig = {
  apiKey: process.env.FIREBASE_API_KEY,
  authDomain: "marketmind-ai.firebaseapp.com",
  projectId: "marketmind-ai",
  storageBucket: "marketmind-ai.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdefghijklmnop"
};

// Inicializar Firebase
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// Configuração OpenAI
const openaiConfig = new configuration({
  apiKey: process.env.OPENAI_API_KEY
});
const openai = new OpenAIApi(openaiConfig);

// Interface para Briefing de Marketing
interface MarketingBriefing {
  empresa: string;
  segmento: string;
  tom: 'profissional' | 'descontraído' | 'inspirador';
  objetivo: 'vendas' | 'engajamento' | 'marca';
  palavrasChave: string[];
}

// Classe de Geração de Conteúdo
class MarketMindAI {
  // Gerar Título de Campanha
  async gerarTitulo(briefing: MarketingBriefing): Promise<string> {
    const prompt = `Crie um título criativo para campanha de marketing de ${briefing.empresa} 
    no segmento de ${briefing.segmento}, com tom ${briefing.tom} 
    e foco em ${briefing.objetivo}. 
    Palavras-chave: ${briefing.palavrasChave.join(', ')}`;

    const response = await openai.createCompletion({
      model: "text-davinci-003",
      prompt: prompt,
      max_tokens: 50
    });

    return response.data.choices[0].text.trim();
  }

  // Gerar Descrição de Produto
  async gerarDescricaoProduto(briefing: MarketingBriefing): Promise<string> {
    const prompt = `Escreva uma descrição persuasiva de produto para ${briefing.empresa}, 
    destacando benefícios únicos, com tom ${briefing.tom}`;

    const response = await openai.createCompletion({
      model: "text-davinci-003",
      prompt: prompt,
      max_tokens: 150
    });

    return response.data.choices[0].text.trim();
  }

  // Gerar Post para Redes Sociais
  async gerarPostSocial(briefing: MarketingBriefing): Promise<string> {
    const prompt = `Crie um post para redes sociais para ${briefing.empresa}, 
    com ${briefing.tom} e objetivo de ${briefing.objetivo}. 
    Use palavras-chave: ${briefing.palavrasChave.join(', ')}`;

    const response = await openai.createCompletion({
      model: "text-davinci-003",
      prompt: prompt,
      max_tokens: 280
    });

    return response.data.choices[0].text.trim();
  }

  // Salvar Conteúdo no Firebase
  async salvarConteudo(
    tipo: 'titulo' | 'descricao' | 'post', 
    conteudo: string, 
    briefing: MarketingBriefing
  ) {
    try {
      const docRef = await addDoc(collection(db, "conteudos_gerados"), {
        tipo,
        conteudo,
        empresa: briefing.empresa,
        segmento: briefing.segmento,
        dataGeracao: new Date()
      });
      console.log("Conteúdo salvo com ID: ", docRef.id);
    } catch (e) {
      console.error("Erro ao salvar conteúdo: ", e);
    }
  }
}

// Exemplo de Uso
async function exemploDemostracao() {
  const marketMind = new MarketMindAI();
  
  const briefing: MarketingBriefing = {
    empresa: "TechInove Soluções",
    segmento: "Tecnologia",
    tom: "profissional",
    objetivo: "vendas",
    palavrasChave: ["inovação", "tecnologia", "transformação digital"]
  };

  // Gerar e salvar conteúdos
  const titulo = await marketMind.gerarTitulo(briefing);
  await marketMind.salvarConteudo('titulo', titulo, briefing);

  const descricao = await marketMind.gerarDescricaoProduto(briefing);
  await marketMind.salvarConteudo('descricao', descricao, briefing);

  const post = await marketMind.gerarPostSocial(briefing);
  await marketMind.salvarConteudo('post', post, briefing);
}

// Iniciar demonstração
exemploDemostracao();

export default MarketMindAI;
