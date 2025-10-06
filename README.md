% ====================================
%   SEÇÃO 6.3 - TESTE COMPLETO
%   Aplicação de todos os efeitos em:
%   1. Melodia da Parte 1
%   2. Guitarra limpa (guitar.wav)
%   3. Voz limpa (voz.wav)
% ====================================

clear all; close all; clc;

fs = 48000;

fprintf('====================================\n');
fprintf('   TESTE COMPLETO - SEÇÃO 6.3\n');
fprintf('====================================\n\n');

% ====================================
%   PREPARAÇÃO DOS SINAIS
% ====================================

fprintf('--- CARREGANDO/GERANDO SINAIS ---\n\n');

% 1. MELODIA DA PARTE 1 (Sinal Sintético)
fprintf('1. Criando melodia da Parte 1...\n');
C = 261.63; E = 329.63; F = 349.23; G = 392.00;
duracao_nota = 0.5;

% Melodia simples: C - E - F - G - F - E - C
melodia = [geraNota(C*2, fs, duracao_nota, 'seno'), ...
           geraNota(E*2, fs, duracao_nota, 'seno'), ...
           geraNota(F*2, fs, duracao_nota, 'seno'), ...
           geraNota(G*2, fs, duracao_nota, 'seno'), ...
           geraNota(F*2, fs, duracao_nota, 'seno'), ...
           geraNota(E*2, fs, duracao_nota, 'seno'), ...
           geraNota(C*2, fs, duracao_nota, 'seno')];

% Normaliza
melodia = melodia / max(abs(melodia)) * 0.8;
fprintf('   Melodia criada: 7 notas, %.1f segundos\n', length(melodia)/fs);

% 2. GUITARRA LIMPA
try
    [guitar, fs_g] = audioread('guitar.wav');
    fprintf('2. Carregando guitar.wav...\n');
    
    % Reamostra se necessário
    if fs_g ~= fs
        guitar = resample(guitar, fs, fs_g);
        fprintf('   Reamostrado de %d Hz para %d Hz\n', fs_g, fs);
    end
    
    % Converte para mono se estéreo
    if size(guitar, 2) > 1
        guitar = mean(guitar, 2)';
        fprintf('   Convertido para mono\n');
    else
        guitar = guitar';
    end
    
    % Normaliza
    guitar = guitar / max(abs(guitar)) * 0.8;
    fprintf('   Guitarra carregada: %.1f segundos\n', length(guitar)/fs);
    
catch
    fprintf('2. AVISO: guitar.wav não encontrado!\n');
    fprintf('   Usando melodia sintética como substituto\n');
    guitar = melodia;
end

% 3. VOZ LIMPA
try
    [voz, fs_v] = audioread('voz.wav');
    fprintf('3. Carregando voz.wav...\n');
    
    if fs_v ~= fs
        voz = resample(voz, fs, fs_v);
        fprintf('   Reamostrado de %d Hz para %d Hz\n', fs_v, fs);
    end
    
    if size(voz, 2) > 1
        voz = mean(voz, 2)';
        fprintf('   Convertido para mono\n');
    else
        voz = voz';
    end
    
    voz = voz / max(abs(voz)) * 0.8;
    fprintf('   Voz carregada: %.1f segundos\n', length(voz)/fs);
    
catch
    fprintf('3. AVISO: voz.wav não encontrado!\n');
    fprintf('   Usando melodia sintética como substituto\n');
    voz = melodia;
end

fprintf('\n');

% ====================================
%   DEFINIÇÃO DOS EFEITOS
% ====================================

fprintf('--- CRIANDO RESPOSTAS AO IMPULSO ---\n\n');

% EFEITO 1: REVERB COM RUÍDO
fprintf('Efeito 1: Reverb com Ruído\n');
alpha_reverb = 0.005;
N_reverb = round(0.5 * fs);
var_ruido = 0.3;
n = 0:N_reverb-1;
h_reverb = exp(-alpha_reverb * n) .* (var_ruido * randn(1, N_reverb));
h_reverb = h_reverb / max(abs(h_reverb));
fprintf('   Duração: 0.5s, Decaimento: %.4f\n', alpha_reverb);

% EFEITO 2: DELAY/ECO
fprintf('Efeito 2: Delay/Eco\n');
D_delay = round(0.25 * fs);  % 250ms
g_delay = 0.5;
num_ecos = 4;
h_delay = zeros(1, (num_ecos + 1) * D_delay);
for k = 0:num_ecos
    h_delay(k * D_delay + 1) = g_delay^k;
end
fprintf('   Atraso: 250ms, Ganho: %.1f, Ecos: %d\n', g_delay, num_ecos);

% EFEITO 3: RESSONÂNCIA METÁLICA
fprintf('Efeito 3: Ressonância Metálica\n');
f_metal = 1200;
decay_metal = 0.01;
dur_metal = 0.3;
n_metal = 0:round(dur_metal * fs) - 1;
h_metal = sin(2*pi*f_metal*n_metal/fs) .* exp(-decay_metal * n_metal);
h_metal = h_metal / max(abs(h_metal));
fprintf('   Frequência: %d Hz, Duração: %.1fs\n', f_metal, dur_metal);

% EFEITO 4: TUBO RESSONANTE
fprintf('Efeito 4: Tubo Ressonante\n');
harmonicos = [200, 400, 600, 800];
dur_tubo = 0.4;
h_tubo = zeros(1, round(dur_tubo * fs));
for f_h = harmonicos
    h_tubo = h_tubo + 0.25 * sin(2*pi*f_h*(0:length(h_tubo)-1)/fs) .* ...
             exp(-0.008 * (0:length(h_tubo)-1));
end
h_tubo = h_tubo / max(abs(h_tubo));
fprintf('   Harmônicos: %s Hz\n', mat2str(harmonicos));

% EFEITO 5: CAIXA PERCUSSIVA
fprintf('Efeito 5: Caixa Percussiva\n');
h_caixa = [1, zeros(1, round(0.05*fs)-1)];
for i = 1:5
    pos = round(0.05*fs*i);
    if pos <= length(h_caixa)
        h_caixa(pos) = 0.6^i;
    end
end
ruido_caixa = 0.3 * randn(1, round(0.3*fs)) .* exp(-0.02*(0:round(0.3*fs)-1));
h_caixa = [h_caixa, ruido_caixa];
h_caixa = h_caixa / max(abs(h_caixa));
fprintf('   Impulsos + ruído decrescente\n');

fprintf('\n');

% ====================================
%   APLICAÇÃO DOS EFEITOS
% ====================================

fprintf('--- APLICANDO EFEITOS ---\n\n');

% Função auxiliar para aplicar e normalizar
aplicar_efeito = @(sinal, h) deal(conv(sinal, h, 'same') / max(abs(conv(sinal, h, 'same'))) * 0.8);

fprintf('Processando MELODIA...\n');
melodia_reverb = aplicar_efeito(melodia, h_reverb);
melodia_delay = aplicar_efeito(melodia, h_delay);
melodia_metal = aplicar_efeito(melodia, h_metal);
melodia_tubo = aplicar_efeito(melodia, h_tubo);
melodia_caixa = aplicar_efeito(melodia, h_caixa);
fprintf('   5 efeitos aplicados\n');

fprintf('Processando GUITARRA...\n');
guitar_reverb = aplicar_efeito(guitar, h_reverb);
guitar_delay = aplicar_efeito(guitar, h_delay);
guitar_metal = aplicar_efeito(guitar, h_metal);
guitar_tubo = aplicar_efeito(guitar, h_tubo);
guitar_caixa = aplicar_efeito(guitar, h_caixa);
fprintf('   5 efeitos aplicados\n');

fprintf('Processando VOZ...\n');
voz_reverb = aplicar_efeito(voz, h_reverb);
voz_delay = aplicar_efeito(voz, h_delay);
voz_metal = aplicar_efeito(voz, h_metal);
voz_tubo = aplicar_efeito(voz, h_tubo);
voz_caixa = aplicar_efeito(voz, h_caixa);
fprintf('   5 efeitos aplicados\n\n');

% ====================================
%   SALVAMENTO DOS ÁUDIOS
% ====================================

fprintf('--- SALVANDO ARQUIVOS DE ÁUDIO ---\n\n');

% Melodia
audiowrite('melodia_original.wav', melodia, fs);
audiowrite('melodia_reverb.wav', melodia_reverb, fs);
audiowrite('melodia_delay.wav', melodia_delay, fs);
audiowrite('melodia_metal.wav', melodia_metal, fs);
audiowrite('melodia_tubo.wav', melodia_tubo, fs);
audiowrite('melodia_caixa.wav', melodia_caixa, fs);
fprintf('Melodia: 6 arquivos salvos\n');

% Guitarra
audiowrite('guitar_original.wav', guitar, fs);
audiowrite('guitar_reverb.wav', guitar_reverb, fs);
audiowrite('guitar_delay.wav', guitar_delay, fs);
audiowrite('guitar_metal.wav', guitar_metal, fs);
audiowrite('guitar_tubo.wav', guitar_tubo, fs);
audiowrite('guitar_caixa.wav', guitar_caixa, fs);
fprintf('Guitarra: 6 arquivos salvos\n');

% Voz
audiowrite('voz_original.wav', voz, fs);
audiowrite('voz_reverb.wav', voz_reverb, fs);
audiowrite('voz_delay.wav', voz_delay, fs);
audiowrite('voz_metal.wav', voz_metal, fs);
audiowrite('voz_tubo.wav', voz_tubo, fs);
audiowrite('voz_caixa.wav', voz_caixa, fs);
fprintf('Voz: 6 arquivos salvos\n\n');

% ====================================
%   VISUALIZAÇÃO COMPARATIVA
% ====================================

fprintf('--- GERANDO VISUALIZAÇÕES ---\n\n');

% FIGURA 1: Comparação Melodia
figure('Name', 'Comparação: MELODIA', 'Position', [50, 50, 1400, 900]);

t_mel = (0:length(melodia)-1) / fs;
n_samples = min(round(1*fs), length(melodia));  % Plota até 1 segundo

subplot(3,2,1);
plot(t_mel(1:n_samples), melodia(1:n_samples), 'b', 'LineWidth', 1.5);
title('MELODIA ORIGINAL', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,2);
plot(t_mel(1:n_samples), melodia_reverb(1:n_samples), 'r', 'LineWidth', 1);
title('Com REVERB', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,3);
plot(t_mel(1:n_samples), melodia_delay(1:n_samples), 'm', 'LineWidth', 1);
title('Com DELAY', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,4);
plot(t_mel(1:n_samples), melodia_metal(1:n_samples), 'g', 'LineWidth', 1);
title('Com RESSONÂNCIA METÁLICA', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,5);
plot(t_mel(1:n_samples), melodia_tubo(1:n_samples), 'c', 'LineWidth', 1);
title('Com TUBO RESSONANTE', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,6);
plot(t_mel(1:n_samples), melodia_caixa(1:n_samples), 'k', 'LineWidth', 1);
title('Com CAIXA PERCUSSIVA', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

% FIGURA 2: Comparação Guitarra
figure('Name', 'Comparação: GUITARRA', 'Position', [100, 100, 1400, 900]);

t_git = (0:length(guitar)-1) / fs;
n_samples_git = min(round(2*fs), length(guitar));

subplot(3,2,1);
plot(t_git(1:n_samples_git), guitar(1:n_samples_git), 'b', 'LineWidth', 1.5);
title('GUITARRA ORIGINAL', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,2);
plot(t_git(1:n_samples_git), guitar_reverb(1:n_samples_git), 'r', 'LineWidth', 1);
title('Com REVERB', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,3);
plot(t_git(1:n_samples_git), guitar_delay(1:n_samples_git), 'm', 'LineWidth', 1);
title('Com DELAY', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,4);
plot(t_git(1:n_samples_git), guitar_metal(1:n_samples_git), 'g', 'LineWidth', 1);
title('Com RESSONÂNCIA METÁLICA', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,5);
plot(t_git(1:n_samples_git), guitar_tubo(1:n_samples_git), 'c', 'LineWidth', 1);
title('Com TUBO RESSONANTE', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,6);
plot(t_git(1:n_samples_git), guitar_caixa(1:n_samples_git), 'k', 'LineWidth', 1);
title('Com CAIXA PERCUSSIVA', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

% FIGURA 3: Comparação Voz
figure('Name', 'Comparação: VOZ', 'Position', [150, 150, 1400, 900]);

t_voz = (0:length(voz)-1) / fs;
n_samples_voz = min(round(2*fs), length(voz));

subplot(3,2,1);
plot(t_voz(1:n_samples_voz), voz(1:n_samples_voz), 'b', 'LineWidth', 1.5);
title('VOZ ORIGINAL', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,2);
plot(t_voz(1:n_samples_voz), voz_reverb(1:n_samples_voz), 'r', 'LineWidth', 1);
title('Com REVERB', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,3);
plot(t_voz(1:n_samples_voz), voz_delay(1:n_samples_voz), 'm', 'LineWidth', 1);
title('Com DELAY', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,4);
plot(t_voz(1:n_samples_voz), voz_metal(1:n_samples_voz), 'g', 'LineWidth', 1);
title('Com RESSONÂNCIA METÁLICA', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,5);
plot(t_voz(1:n_samples_voz), voz_tubo(1:n_samples_voz), 'c', 'LineWidth', 1);
title('Com TUBO RESSONANTE', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

subplot(3,2,6);
plot(t_voz(1:n_samples_voz), voz_caixa(1:n_samples_voz), 'k', 'LineWidth', 1);
title('Com CAIXA PERCUSSIVA', 'FontWeight', 'bold');
xlabel('Tempo (s)'); ylabel('Amplitude'); grid on; ylim([-1 1]);

fprintf('3 figuras criadas com comparações visuais\n\n');

% ====================================
%   ANÁLISE E DISCUSSÃO
% ====================================

fprintf('====================================\n');
fprintf('   ANÁLISE DOS RESULTADOS\n');
fprintf('====================================\n\n');

fprintf('OBSERVAÇÕES GERAIS:\n\n');

fprintf('1. REVERB (Sala Virtual):\n');
fprintf('   - Melodia: Adiciona "espaço" ao som sintético\n');
fprintf('   - Guitarra: Simula ambiente de gravação\n');
fprintf('   - Voz: Cria sensação de profundidade\n');
fprintf('   → Melhor para: Voz e instrumentos acústicos\n\n');

fprintf('2. DELAY (Eco):\n');
fprintf('   - Melodia: Cria repetições rítmicas\n');
fprintf('   - Guitarra: Efeito clássico de guitarras\n');
fprintf('   - Voz: Interessante para refrões\n');
fprintf('   → Melhor para: Guitarra e efeitos rítmicos\n\n');

fprintf('3. RESSONÂNCIA METÁLICA:\n');
fprintf('   - Melodia: Adiciona timbre brilhante\n');
fprintf('   - Guitarra: Som "robótico" ou eletrônico\n');
fprintf('   - Voz: Efeito de "telefone" ou rádio\n');
fprintf('   → Melhor para: Efeitos especiais\n\n');

fprintf('4. TUBO RESSONANTE:\n');
fprintf('   - Melodia: Som "oco" ou distante\n');
fprintf('   - Guitarra: Adiciona harmônicos graves\n');
fprintf('   - Voz: Efeito de megafone\n');
fprintf('   → Melhor para: Voz e efeitos criativos\n\n');

fprintf('5. CAIXA PERCUSSIVA:\n');
fprintf('   - Melodia: Adiciona textura ruidosa\n');
fprintf('   - Guitarra: Som de "arranhado"\n');
fprintf('   - Voz: Efeito "granular"\n');
fprintf('   → Melhor para: Percussão e texturas\n\n');

fprintf('====================================\n');
fprintf('   TESTE CONCLUÍDO!\n');
fprintf('====================================\n\n');

fprintf('Arquivos gerados:\n');
fprintf('  - 18 arquivos .wav (3 sinais × 6 versões cada)\n');
fprintf('  - 3 figuras com comparações visuais\n\n');

fprintf('Para ouvir os resultados, use:\n');
fprintf('  sound(melodia_reverb, fs)\n');
fprintf('  sound(guitar_delay, fs)\n');
fprintf('  sound(voz_metal, fs)\n\n');

% ====================================
%   FUNÇÃO AUXILIAR
% ====================================

function nota = geraNota(f0, fs, duracao, tipo)
    t = 0:1/fs:duracao;
    switch tipo
        case 'seno'
            nota = sin(2*pi*f0*t);
        case 'quadrada'
            nota = square(2*pi*f0*t);
        case 'serra'
            nota = sawtooth(2*pi*f0*t, 1);
        case 'triangular'
            nota = sawtooth(2*pi*f0*t, 0.5);
        otherwise
            error('Tipo desconhecido');
    end
end
