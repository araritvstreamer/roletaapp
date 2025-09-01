// PromptForge - Suporte a múltiplos templates por nicho

import { useState } from 'react';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Card, CardContent } from '@/components/ui/card';
import { motion } from 'framer-motion';

export default function Home() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [email, setEmail] = useState('');
  const [senha, setSenha] = useState('');
  const [isSubscribed, setIsSubscribed] = useState(false);
  const [step, setStep] = useState(1);
  const [savedProjects, setSavedProjects] = useState([]);
  const [selectedNiche, setSelectedNiche] = useState('default');
  const [formData, setFormData] = useState({
    nomeProjeto: '',
    problemaResolvido: '',
    publicoAlvo: '',
    funcionalidades: '',
    referenciasVisuais: '',
    estiloVisual: '',
    fontePrincipal: '',
  });

  const isAdmin = email === 'admin@promptforge.com';

  const handleLogin = () => {
    if (email && senha) {
      setIsLoggedIn(true);
      setIsSubscribed(email === 'admin@promptforge.com' ? true : false);
    }
  };

  const handleSubscribe = () => {
    alert('🔗 Redirecionando para checkout Stripe (simulado)');
    setIsSubscribed(true);
  };

  const handleChange = (field, value) => {
    setFormData({ ...formData, [field]: value });
  };

  const nextStep = () => setStep((s) => s + 1);
  const prevStep = () => setStep((s) => s - 1);

  const niches = {
    default: (data) => `🧠 Prompt Estratégico\n\n🔹 Projeto: ${data.nomeProjeto}\n🔹 Problema resolvido: ${data.problemaResolvido}\n🔹 Público-alvo: ${data.publicoAlvo}\n🔹 Funcionalidades: ${data.funcionalidades}\n🔹 Visual: ${data.estiloVisual} com fonte ${data.fontePrincipal}\n🔹 Referência visual: ${data.referenciasVisuais}`,

    barbearia: (data) => `Crie um sistema de agendamento de horários para uma barbearia chamado ${data.nomeProjeto}.\nO sistema deve resolver o problema: ${data.problemaResolvido}.\nO público são: ${data.publicoAlvo}.\nFuncionalidades: ${data.funcionalidades}.\nUse o estilo visual ${data.estiloVisual}, com a fonte ${data.fontePrincipal}.\nInspiração visual: ${data.referenciasVisuais}.`,

    clinica: (data) => `Construa um painel de marcações online para uma clínica estética, nome: ${data.nomeProjeto}.\nProblema resolvido: ${data.problemaResolvido}.\nPara: ${data.publicoAlvo}.\nInclui: ${data.funcionalidades}.\nDesign ${data.estiloVisual}, fonte ${data.fontePrincipal}.\nReferências: ${data.referenciasVisuais}`
  };

  const generatePrompt = () => {
    const builder = niches[selectedNiche] || niches.default;
    return builder(formData);
  };

  const saveProject = () => {
    const prompt = generatePrompt();
    setSavedProjects([...savedProjects, { ...formData, prompt, niche: selectedNiche }]);
    alert('✅ Projeto salvo com sucesso!');
    setFormData({
      nomeProjeto: '',
      problemaResolvido: '',
      publicoAlvo: '',
      funcionalidades: '',
      referenciasVisuais: '',
      estiloVisual: '',
      fontePrincipal: '',
    });
    setStep(1);
  };

  if (!isLoggedIn) {
    return (
      <main className="min-h-screen flex items-center justify-center p-4">
        <Card className="w-full max-w-sm">
          <CardContent className="p-6 space-y-4">
            <h2 className="text-xl font-bold">Acessar PromptForge</h2>
            <Input placeholder="Seu e-mail" value={email} onChange={(e) => setEmail(e.target.value)} />
            <Input type="password" placeholder="Senha" value={senha} onChange={(e) => setSenha(e.target.value)} />
            <Button className="w-full" onClick={handleLogin}>Entrar</Button>
          </CardContent>
        </Card>
      </main>
    );
  }

  if (!isSubscribed && !isAdmin) {
    return (
      <main className="min-h-screen flex items-center justify-center p-4">
        <Card className="w-full max-w-md">
          <CardContent className="p-6 space-y-4">
            <h2 className="text-xl font-bold">Assine para usar o PromptForge</h2>
            <p>Este aplicativo é exclusivo para assinantes. Acesse prompts ilimitados com apenas R$29/mês.</p>
            <Button className="w-full" onClick={handleSubscribe}>Assinar Agora</Button>
          </CardContent>
        </Card>
      </main>
    );
  }

  if (isAdmin) {
    return (
      <main className="min-h-screen p-4">
        <h1 className="text-2xl font-bold mb-4">Painel do Admin 👑</h1>
        <p className="mb-2">Total de projetos gerados: {savedProjects.length}</p>
        <div className="space-y-4">
          {savedProjects.map((proj, i) => (
            <Card key={i}>
              <CardContent className="p-4">
                <p className="text-sm text-muted-foreground mb-2">[{proj.niche}] {proj.nomeProjeto}</p>
                <pre className="text-xs whitespace-pre-wrap">{proj.prompt}</pre>
              </CardContent>
            </Card>
          ))}
        </div>
      </main>
    );
  }

  const nicheOptions = [
    { value: 'default', label: 'Genérico' },
    { value: 'barbearia', label: 'Barbearia' },
    { value: 'clinica', label: 'Clínica Estética' }
  ];

  return (
    <main className="min-h-screen p-4 flex flex-col items-center justify-center bg-background">
      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        className="w-full max-w-xl space-y-4"
      >
        <Card>
          <CardContent className="p-6 space-y-4">
            <h2 className="text-xl font-bold">Etapa {step} de {steps.length}</h2>
            <div>
              <label className="text-sm font-medium">Escolha o nicho do projeto</label>
              <select
                className="w-full border rounded p-2 mt-1"
                value={selectedNiche}
                onChange={(e) => setSelectedNiche(e.target.value)}
              >
                {nicheOptions.map((opt) => (
                  <option key={opt.value} value={opt.value}>{opt.label}</option>
                ))}
              </select>
            </div>
            <div>
              <p className="mb-2 font-medium">{steps[step - 1].label}</p>
              <Input
                placeholder={steps[step - 1].placeholder}
                value={formData[steps[step - 1].field]}
                onChange={(e) => handleChange(steps[step - 1].field, e.target.value)}
              />
            </div>
            <div className="flex justify-between">
              <Button onClick={prevStep} disabled={step === 1}>Voltar</Button>
              {step < steps.length ? (
                <Button onClick={nextStep}>Próxima</Button>
              ) : (
                <Button onClick={saveProject}>Salvar e Gerar Prompt</Button>
              )}
            </div>
          </CardContent>
        </Card>

        <div className="w-full">
          <h3 className="text-lg font-semibold mb-2">Seus projetos</h3>
          {savedProjects.map((proj, i) => (
            <Card key={i} className="mb-2">
              <CardContent className="p-4">
                <p className="font-medium text-sm">[{proj.niche}] {proj.nomeProjeto}</p>
                <pre className="text-xs whitespace-pre-wrap mt-2">{proj.prompt}</pre>
              </CardContent>
            </Card>
          ))}
        </div>
      </motion.div>
    </main>
  );
}

