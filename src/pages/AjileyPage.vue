<template>
  <q-page class="flex flex-center text-white q-pa-xs ajiley-page">
    <div
      class="game-container column items-center q-gutter-xs q-pa-sm q-pa-md-md relative-position"
      style="max-width: 750px; width: 100%"
    >
      <!-- Grade de Pontuação -->
      <div class="score-grid-box">
        <table class="score-table">
          <thead>
            <tr>
              <th>Jog.</th>
              <th>1</th>
              <th>2</th>
              <th>3</th>
              <th v-if="historicoHumano.length > 3">4</th>
              <th>TOT</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td><b>Tu</b></td>
              <td>{{ historicoHumano[0] || '-' }}</td>
              <td>{{ historicoHumano[1] || '-' }}</td>
              <td>{{ historicoHumano[2] || '-' }}</td>
              <td v-if="historicoHumano.length > 3">{{ historicoHumano[3] || '-' }}</td>
              <td>
                <b>{{ totalHumano }}</b>
              </td>
            </tr>
            <tr>
              <td><b>IA</b></td>
              <td>{{ historicoIA[0] || '-' }}</td>
              <td>{{ historicoIA[1] || '-' }}</td>
              <td>{{ historicoIA[2] || '-' }}</td>
              <td v-if="historicoIA.length > 3">{{ historicoIA[3] || '-' }}</td>
              <td>
                <b>{{ totalIA }}</b>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pozo Acumulado -->
      <div class="table-pot-display-top row items-center q-px-sm q-py-xs">
        <q-icon name="casino" color="amber-4" size="xs" class="q-mr-xs" />
        <span class="text-caption text-weight-bold text-amber-3">Pozo:</span>
        <span class="text-caption text-weight-bold text-white q-ml-xs">{{ pote }} Fichas</span>
      </div>

      <!-- Encabezado / Título y Marcador -->
      <div
        class="column items-center q-gutter-xs w-full q-px-xs top-header-section"
        style="width: 100%"
      >
        <div class="row items-center justify-between w-full q-px-sm">
          <div class="text-subtitle1 text-weight-bold text-white tracking-wide text-shadow">
            Ajiley Game Master (Poker 51)
          </div>
          <q-badge color="amber-9" class="text-caption text-weight-bold q-px-sm">
            Mano: {{ maoAtual }} / {{ historicoHumano.length > 3 ? '4 (Desempate)' : '3' }}
          </q-badge>
        </div>

        <div class="row justify-center items-center q-gutter-sm q-my-none">
          <q-chip class="game-chip dense" icon="person">Tus Fichas: {{ fichasHumano }}</q-chip>
          <q-chip class="game-chip dense" icon="smart_toy">Fichas IA: {{ fichasIA }}</q-chip>
        </div>

        <div class="row justify-center items-center q-gutter-xs flex-wrap q-mt-xs">
          <q-btn
            color="amber-9"
            text-color="dark"
            icon="play_arrow"
            label="Nueva Partida"
            @click="reiniciarPartidaCompleta"
            class="action-btn text-weight-bold"
            dense
            unelevated
          />
          <q-btn
            color="cyan-8"
            icon="info"
            label="Regras"
            @click="mostrarRegras = true"
            class="action-btn text-weight-bold"
            dense
            unelevated
          />
          <q-btn
            color="negative"
            icon="exit_to_app"
            label="Salir"
            @click="exitGame"
            class="action-btn text-weight-bold"
            dense
            unelevated
          />
        </div>
      </div>

      <div class="status-banner text-amber-3 text-center q-px-sm q-py-xs shadow-1">
        {{ mensagemStatus }}
      </div>

      <!-- Tapete de Juego (Mesa) -->
      <div
        class="game-table-felt column items-center q-gutter-sm q-pa-sm shadow-10"
        style="width: 100%"
      >
        <div v-if="temAjileyInstantaneo" class="ajiley-alert-box q-pa-sm text-center q-my-xs">
          <div
            class="text-h6 text-yellow-3 text-weight-bold uppercase tracking-widest text-shadow-glow"
          >
            ⭐ ¡¡¡ AJILEY 51 EXACTOS !!! ⭐
          </div>
          <div class="text-caption text-white">{{ mensajeAjileyInstantaneo }}</div>
        </div>

        <div class="column items-center">
          <div class="text-caption text-blue-2 q-mb-none tracking-wider">
            Oponente (IA) — Cartas: {{ maoAI.length }}
          </div>
          <div class="row q-gutter-xs justify-center">
            <div
              v-for="(carta, idx) in maoAI"
              :key="'ai-' + idx"
              :class="
                faseJogo === 'SHOWDOWN' || faseJogo === 'FIN_PARTIDA' || temAjileyInstantaneo
                  ? 'player-card-compact flex flex-center'
                  : 'table-card-back-compact flex flex-center'
              "
            >
              <img
                v-if="faseJogo === 'SHOWDOWN' || faseJogo === 'FIN_PARTIDA' || temAjileyInstantaneo"
                src="/images/cartasTruco.png"
                class="atlas-img"
                :style="getSpriteStyle(carta)"
              />
              <span v-else class="text-subtitle1 text-amber-4">🂠</span>
            </div>
          </div>
        </div>

        <div class="row justify-center items-center w-full q-py-xs">
          <div
            v-if="faseJogo === 'SHOWDOWN'"
            class="column items-center q-pa-xs results-table-banner"
          >
            <div class="text-subtitle2 text-amber-4 text-weight-bold">
              🏆 Resultado de la Mano {{ maoAtual }}
            </div>
            <div
              class="text-caption text-center text-white q-my-xs"
              v-html="mensagemResultado"
            ></div>
            <q-btn
              color="amber-7"
              text-color="dark"
              icon="navigate_next"
              :label="
                maoAtual === 3 && totalHumano === totalIA
                  ? 'Jugar Mano de Desempate'
                  : 'Siguiente Mano'
              "
              @click="iniciarProximaMano"
              class="text-weight-bold q-px-sm"
              size="sm"
              dense
              unelevated
            />
          </div>

          <div
            v-else-if="faseJogo === 'FIN_PARTIDA'"
            class="column items-center q-pa-xs results-table-banner"
            :class="totalHumano > totalIA ? 'win-glory-box' : 'win-glow-box'"
          >
            <div class="text-subtitle2 text-amber-4 text-weight-bold">👑 ¡Partida Finalizada!</div>
            <div
              class="text-caption text-center text-white q-my-xs"
              v-html="mensagemResultadoFinal"
            ></div>
            <q-btn
              color="positive"
              icon="replay"
              label="Jugar Nueva Partida"
              @click="reiniciarPartidaCompleta"
              class="text-weight-bold q-px-sm"
              size="sm"
              dense
              unelevated
            />
          </div>

          <template v-else>
            <div class="row items-center justify-center q-gutter-md w-full">
              <div class="row items-center q-gutter-xs">
                <span class="text-caption text-amber-3 text-weight-bold">Fase:</span>
                <q-badge
                  style="border-radius: 6px"
                  color="orange-8"
                  class="text-caption q-px-sm q-py-xs"
                  >{{ faseAtualTexto }}</q-badge
                >
              </div>

              <div
                v-if="
                  (faseJogo === 'APUESTA_1' ||
                    faseJogo === 'APUESTA_2' ||
                    faseJogo === 'APUESTA_3') &&
                  !turnoAguardandoIA
                "
                class="row q-gutter-xs items-center justify-center action-buttons-group"
              >
                <q-btn
                  style="border-radius: 6px"
                  color="positive"
                  label="Apostar / Igualar (10)"
                  @click="apostar(10)"
                  size="sm"
                  unelevated
                />
                <q-btn
                  style="border-radius: 6px"
                  color="amber-8"
                  text-color="dark"
                  label="Duplicar Apuesta (20)"
                  @click="duplicarApuesta"
                  size="sm"
                  unelevated
                />
                <q-btn
                  style="border-radius: 6px"
                  color="negative"
                  label="Pasar"
                  @click="passarRodada"
                  size="sm"
                  unelevated
                />
              </div>

              <div v-if="turnoAguardandoIA" class="text-caption text-amber-4">
                🤖 La IA está pensando su jugada...
              </div>

              <div v-if="faseJogo === 'DESCARTE'" class="row q-gutter-xs items-center">
                <span class="text-caption text-yellow-2">Selecciona cartas:</span>
                <q-btn
                  color="amber-7"
                  text-color="dark"
                  label="Confirmar Descarte"
                  @click="executarDescarte"
                  class="text-weight-bold"
                  dense
                  size="sm"
                  unelevated
                />
              </div>
            </div>
          </template>
        </div>

        <div class="column items-center">
          <div class="text-caption text-blue-2 q-mb-none tracking-wider">
            Tu Mano
            <span v-if="faseJogo === 'DESCARTE'">(Haz clic para cambiar)</span>
          </div>
          <div class="row q-gutter-xs justify-center player-cards-container">
            <div
              v-for="(carta, index) in maoHumano"
              :key="'humano-' + index"
              class="player-card-compact flex flex-center"
              :class="{
                'selected-card': cartasSelecionadasDescarte.includes(index),
                'disabled-card': faseJogo !== 'DESCARTE',
              }"
              @click="toggleSelecionarCarta(index)"
            >
              <img src="/images/cartasTruco.png" class="atlas-img" :style="getSpriteStyle(carta)" />
            </div>
          </div>
        </div>
      </div>

      <q-dialog v-model="mostrarRegras">
        <q-card
          class="bg-dark text-white q-pa-md dialog-effect"
          style="max-width: 450px; width: 90vw"
        >
          <q-card-section>
            <div class="text-subtitle1 text-amber-4 text-weight-bold q-mb-sm">
              Regras del Ajiley (51)
            </div>
            <p class="text-grey-3 text-caption">
              • <b>Objetivo:</b> Sumar la mayor puntuación posible con cartas del mismo palo.<br />
              • <b>Comodín:</b> El 7 de oros actúa como comodín sumando 10 puntos al mejor palo.<br />
              • <b>Ajiley Automático:</b> Si un jugador consigue exactamente 51 puntos, canta
              ¡Ajiley! y gana la mano de forma instantánea con efectos especiales.<br />
              • <b>Duplicar Apuesta:</b> Puedes arriesgar el doble (20 fichas por jugador) durante
              las rondas de apuestas para aumentar la tensión y las ganancias.<br />
              • <b>Fase de Descarte:</b> Tras recibir las 5 cartas, puedes seleccionar las cartas
              que não te sirvan y cambiarlas por nuevas del mazo para mejorar tu jugada.<br />
              • <b>Rendición (Pasar):</b> Si consideras que tu mano es muy débil durante las rondas
              de apuestas, puedes retirarte de la mano para evitar perder más fichas en el pozo.<br />
              • <b>Apuestas y Pozo Acumulativo:</b> Las fichas apostadas van directamente al pozo
              central y se acumulan mano a mano sin regresar a los jugadores. El ganador absoluto de
              la partida se lleva todo el pozo acumulado.
            </p>
          </q-card-section>
          <q-card-actions align="right">
            <q-btn
              style="border-radius: 6px"
              color="amber-5"
              text-color="dark"
              label="Entendido"
              v-close-popup
              dense
              size="sm"
              unelevated
            />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import confetti from 'canvas-confetti';

const router = useRouter();

const fichasHumano = ref(300);
const fichasIA = ref(300);
const pote = ref(0);
const maoAtual = ref(1);

const historicoHumano = ref<number[]>([0, 0, 0]);
const historicoIA = ref<number[]>([0, 0, 0]);
const totalHumano = computed(() => historicoHumano.value.reduce((a, b) => a + b, 0));
const totalIA = computed(() => historicoIA.value.reduce((a, b) => a + b, 0));

const faseJogo = ref('INICIAL');
const mensagemStatus = ref('Haz clic en "Nueva Partida" para comenzar El Ajiley.');
const turnoAguardandoIA = ref(false);

const maoHumano = ref<string[]>([]);
const maoAI = ref<string[]>([]);
const cartasSelecionadasDescarte = ref<number[]>([]);
let baralhoAtual: string[] = [];

const mostrarRegras = ref(false);
const mensagemResultado = ref('');
const mensagemResultadoFinal = ref('');

const temAjileyInstantaneo = ref(false);
const mensajeAjileyInstantaneo = ref('');

const naipes = ['copa', 'oro', 'espada', 'basto'];
const valoresValidos = [1, 2, 3, 4, 5, 6, 7, 10, 11, 12];

const faseJogoTextoMap: Record<string, string> = {
  APUESTA_1: '1ª Ronda de Apuestas (10 fichas)',
  APUESTA_2: '2ª Ronda de Apuestas (10 fichas)',
  DESCARTE: 'Fase de Descarte (Cambio)',
  APUESTA_3: '3ª Ronda de Apuestas (10 fichas)',
  SHOWDOWN: 'Conteo Final',
};

const faseAtualTexto = computed(() => faseJogoTextoMap[faseJogo.value] || 'Esperando');

const getSpriteStyle = (carta: string) => {
  if (!carta) return {};
  const partes = carta.split('_');
  const num = parseInt(partes[0] ?? '1', 10);
  const naipe = (partes[1] ?? '').toLowerCase();

  let linha = 0;
  if (naipe === 'copa') linha = 0;
  else if (naipe === 'oro' || naipe === 'ouro') linha = 1;
  else if (naipe === 'espada') linha = 2;
  else if (naipe === 'basto') linha = 3;

  let coluna = num - 1;
  if (num >= 10) coluna = num - 3;

  return {
    transform: `translate(${-coluna * 10}%, ${-linha * 25}%)`,
  };
};

const criarBaralhoAjiley = () => {
  let baralho: string[] = [];
  for (const naipe of naipes) {
    for (const val of valoresValidos) {
      baralho.push(`${val}_${naipe}`);
    }
  }
  return baralho.sort(() => 0.5 - Math.random());
};

const triggerWinEffects = () => {
  const duration = 4.5 * 1000;
  const end = Date.now() + duration;

  (function frame() {
    confetti({
      particleCount: 25,
      angle: 60,
      spread: 80,
      origin: { x: 0, y: 0.7 },
      colors: ['#ffd700', '#ff4500', '#00ffcc', '#ffffff', '#ff69b4'],
    });
    confetti({
      particleCount: 25,
      angle: 120,
      spread: 80,
      origin: { x: 1, y: 0.7 },
      colors: ['#ffd700', '#ff4500', '#00ffcc', '#ffffff', '#ff69b4'],
    });
    if (Date.now() < end) {
      requestAnimationFrame(frame);
    }
  })();
};

const reiniciarPartidaCompleta = () => {
  fichasHumano.value = 300;
  fichasIA.value = 300;
  pote.value = 0;
  maoAtual.value = 1;
  historicoHumano.value = [0, 0, 0];
  historicoIA.value = [0, 0, 0];
  iniciarMao();
};

const iniciarMao = () => {
  cartasSelecionadasDescarte.value = [];
  temAjileyInstantaneo.value = false;

  pote.value += 20;
  fichasHumano.value -= 10;
  fichasIA.value -= 10;

  turnoAguardandoIA.value = false;
  baralhoAtual = criarBaralhoAjiley();

  maoHumano.value = [baralhoAtual.pop()!, baralhoAtual.pop()!];
  maoAI.value = [baralhoAtual.pop()!, baralhoAtual.pop()!];

  faseJogo.value = 'APUESTA_1';
  mensagemStatus.value = `Mano ${maoAtual.value} — Apuesta inicial de 10 fichas al pozo. Analiza tus cartas.`;

  verificarAjileyInstantaneo();
};

const verificarAjileyInstantaneo = () => {
  const resHumano = avaliarMaoAjiley(maoHumano.value);
  const resAI = avaliarMaoAjiley(maoAI.value);

  const idxMao = maoAtual.value - 1;
  if (historicoHumano.value.length <= idxMao) {
    historicoHumano.value.push(0);
    historicoIA.value.push(0);
  }

  if (resHumano.pontos === 51 || resAI.pontos === 51) {
    temAjileyInstantaneo.value = true;
    faseJogo.value = 'SHOWDOWN';

    historicoHumano.value[idxMao] = resHumano.pontos;
    historicoIA.value[idxMao] = resAI.pontos;

    if (resHumano.pontos === 51 && resAI.pontos === 51) {
      mensajeAjileyInstantaneo.value = `¡Ambos consiguieron 51 puntos exactos (Doble Ajiley)! Gana por mano el jugador humano.`;
      mensagemResultado.value = `⭐ <b>¡Ajiley para ambos! Lograste 51 puntos exactos (${resHumano.naipe}).</b>`;
    } else if (resHumano.pontos === 51) {
      mensajeAjileyInstantaneo.value = `¡Cantaste Ajiley con 51 puntos exactos en la mano ${maoAtual.value}!`;
      mensagemResultado.value = `🎉 <b>¡AJILEY! Conseguiste 51 puntos exactos (${resHumano.naipe}). ¡Victoria instantánea de la mano!</b>`;
    } else {
      mensajeAjileyInstantaneo.value = `¡La IA cantó Ajiley con 51 puntos exactos (${resAI.naipe})!`;
      mensagemResultado.value = `🤖 <b>¡AJILEY de la IA! Consiguió 51 puntos exactos (${resAI.naipe}).</b>`;
    }

    triggerWinEffects();

    if (maoAtual.value >= 3 && totalHumano.value !== totalIA.value) {
      setTimeout(() => verificarFimPartida(), 1500);
    } else if (maoAtual.value >= 4) {
      setTimeout(() => verificarFimPartida(), 1500);
    }
  }
};

const iniciarProximaMano = () => {
  maoAtual.value++;
  if (maoAtual.value === 4 && totalHumano.value === totalIA.value) {
    historicoHumano.value.push(0);
    historicoIA.value.push(0);
    iniciarMao();
  } else if (maoAtual.value > 3 && totalHumano.value !== totalIA.value) {
    verificarFimPartida();
  } else if (maoAtual.value > 4) {
    verificarFimPartida();
  } else {
    iniciarMao();
  }
};

const apostar = (valor: number) => {
  pote.value += valor * 2;
  fichasHumano.value -= valor;
  fichasIA.value -= valor;

  turnoAguardandoIA.value = true;
  mensagemStatus.value = '🤖 La IA está evaluando sus cartas...';

  setTimeout(() => {
    turnoAguardandoIA.value = false;
    const resAI = avaliarMaoAjiley(maoAI.value);

    const limiarPuntuacao = faseJogo.value === 'APUESTA_1' ? 10 : 25;
    const decidePassar = resAI.pontos < limiarPuntuacao && Math.random() < 0.4;

    if (decidePassar) {
      mensagemStatus.value = '🤖 La IA consideró que su mano es débil y decidió pasar (retirarse).';

      const idxMao = maoAtual.value - 1;
      if (historicoHumano.value.length <= idxMao) {
        historicoHumano.value.push(0);
        historicoIA.value.push(0);
      }
      const resHumanoTemp = avaliarMaoAjiley(maoHumano.value);
      historicoHumano.value[idxMao] = resHumanoTemp.pontos;
      historicoIA.value[idxMao] = resAI.pontos;

      mensagemResultado.value = `🎉 <b>¡La IA se retiró de la mano!</b>`;

      if (maoAtual.value >= 3 && totalHumano.value !== totalIA.value) {
        verificarFimPartida();
      } else if (maoAtual.value >= 4) {
        verificarFimPartida();
      } else {
        faseJogo.value = 'SHOWDOWN';
      }
      return;
    }

    if (faseJogo.value === 'APUESTA_1') {
      mensagemStatus.value = '🤖 La IA aceptó. Recibiendo 3 cartas adicionales (total 5)...';
      setTimeout(() => {
        maoHumano.value.push(baralhoAtual.pop()!, baralhoAtual.pop()!, baralhoAtual.pop()!);
        maoAI.value.push(baralhoAtual.pop()!, baralhoAtual.pop()!, baralhoAtual.pop()!);

        verificarAjileyInstantaneo();
        if (temAjileyInstantaneo.value) return;

        faseJogo.value = 'APUESTA_2';
        mensagemStatus.value = 'Segunda ronda de apuestas (10 fichas).';
      }, 1000);
    } else if (faseJogo.value === 'APUESTA_2') {
      faseJogo.value = 'DESCARTE';
      mensagemStatus.value = 'Selecciona las cartas que deseas cambiar y confirma.';
    } else if (faseJogo.value === 'APUESTA_3') {
      faseJogo.value = 'SHOWDOWN';
      calcularResultadoFinal();
    }
  }, 1200);
};

const duplicarApuesta = () => {
  const valorDuplicado = 20;
  pote.value += valorDuplicado * 2;
  fichasHumano.value -= valorDuplicado;
  fichasIA.value -= valorDuplicado;

  mensagemStatus.value = '🔥 ¡Apuesta duplicada a 20 fichas por jugador! El pozo arde.';

  turnoAguardandoIA.value = true;
  setTimeout(() => {
    turnoAguardandoIA.value = false;
    const resAI = avaliarMaoAjiley(maoAI.value);

    const decidePassar = resAI.pontos < 30 && Math.random() < 0.6;

    if (decidePassar) {
      mensagemStatus.value = '🤖 La IA se asustó con la apuesta doble y decidió retirarse.';
      const idxMao = maoAtual.value - 1;
      if (historicoHumano.value.length <= idxMao) {
        historicoHumano.value.push(0);
        historicoIA.value.push(0);
      }
      const resHumanoTemp = avaliarMaoAjiley(maoHumano.value);
      historicoHumano.value[idxMao] = resHumanoTemp.pontos;
      historicoIA.value[idxMao] = resAI.pontos;

      mensagemResultado.value = `🎉 <b>¡La IA se retiró tras tu apuesta doble!</b>`;

      if (maoAtual.value >= 3 && totalHumano.value !== totalIA.value) {
        verificarFimPartida();
      } else if (maoAtual.value >= 4) {
        verificarFimPartida();
      } else {
        faseJogo.value = 'SHOWDOWN';
      }
      return;
    }

    if (faseJogo.value === 'APUESTA_1') {
      mensagemStatus.value = '🤖 La IA aceptó el desafío doble. Recibiendo cartas...';
      setTimeout(() => {
        maoHumano.value.push(baralhoAtual.pop()!, baralhoAtual.pop()!, baralhoAtual.pop()!);
        maoAI.value.push(baralhoAtual.pop()!, baralhoAtual.pop()!, baralhoAtual.pop()!);

        verificarAjileyInstantaneo();
        if (temAjileyInstantaneo.value) return;

        faseJogo.value = 'APUESTA_2';
        mensagemStatus.value = 'Segunda ronda de apuestas.';
      }, 1000);
    } else if (faseJogo.value === 'APUESTA_2') {
      faseJogo.value = 'DESCARTE';
      mensagemStatus.value = 'Selecciona las cartas que deseas cambiar y confirma.';
    } else if (faseJogo.value === 'APUESTA_3') {
      faseJogo.value = 'SHOWDOWN';
      calcularResultadoFinal();
    }
  }, 1200);
};

const passarRodada = () => {
  mensagemStatus.value = 'Te retiraste de la mano.';

  const idxMao = maoAtual.value - 1;
  if (historicoHumano.value.length <= idxMao) {
    historicoHumano.value.push(0);
    historicoIA.value.push(0);
  }
  const resHumanoTemp = avaliarMaoAjiley(maoHumano.value);
  const resAITemp = avaliarMaoAjiley(maoAI.value);
  historicoHumano.value[idxMao] = resHumanoTemp.pontos;
  historicoIA.value[idxMao] = resAITemp.pontos;

  mensagemResultado.value = `🤖 Te retiraste de la mano.`;

  if (maoAtual.value >= 3 && totalHumano.value !== totalIA.value) {
    verificarFimPartida();
  } else if (maoAtual.value >= 4) {
    verificarFimPartida();
  } else {
    faseJogo.value = 'SHOWDOWN';
  }
};

const toggleSelecionarCarta = (index: number) => {
  if (faseJogo.value !== 'DESCARTE') return;
  const pos = cartasSelecionadasDescarte.value.indexOf(index);
  if (pos > -1) {
    cartasSelecionadasDescarte.value.splice(pos, 1);
  } else {
    cartasSelecionadasDescarte.value.push(index);
  }
};

const executarDescarte = () => {
  for (const idx of cartasSelecionadasDescarte.value) {
    const novaCarta = baralhoAtual.pop();
    if (novaCarta) {
      maoHumano.value[idx] = novaCarta;
    }
  }
  cartasSelecionadasDescarte.value = [];

  const resAI = avaliarMaoAjiley(maoAI.value);
  maoAI.value = maoAI.value.filter(
    (carta) => carta === '7_oro' || carta === '7_oros' || carta.split('_')[1] === resAI.naipe,
  );
  while (maoAI.value.length < 5) {
    const novaCartaAI = baralhoAtual.pop();
    if (novaCartaAI) maoAI.value.push(novaCartaAI);
  }

  verificarAjileyInstantaneo();
  if (temAjileyInstantaneo.value) return;

  faseJogo.value = 'APUESTA_3';
  mensagemStatus.value = 'Tercera y última ronda de apuestas antes de mostrar cartas.';
};

const calcularPontosCarta = (carta: string) => {
  const val = parseInt(carta.split('_')[0] ?? '0', 10);
  if (val === 1) return 11;
  if (val === 7) return 7;
  if (val === 6) return 6;
  return 10;
};

const avaliarMaoAjiley = (mao: string[]) => {
  const somasPorNaipe: Record<string, number> = { copa: 0, oro: 0, espada: 0, basto: 0 };
  let temComodin = false;

  for (const carta of mao) {
    if (carta === '7_oro' || carta === '7_oros') {
      temComodin = true;
      continue;
    }
    const partes = carta.split('_');
    const naipe = partes[1];
    if (!naipe || somasPorNaipe[naipe] === undefined) continue;
    const pts = calcularPontosCarta(carta);
    somasPorNaipe[naipe] += pts;
  }

  let melhorNaipe = 'copa';
  let maiorSoma = 0;
  for (const n in somasPorNaipe) {
    const soma = somasPorNaipe[n] ?? 0;
    if (soma > maiorSoma) {
      maiorSoma = soma;
      melhorNaipe = n;
    }
  }

  if (temComodin) {
    maiorSoma += 10;
  }

  return { pontos: maiorSoma, naipe: melhorNaipe, temAjiley: maiorSoma === 51 };
};

const calcularResultadoFinal = () => {
  const resHumano = avaliarMaoAjiley(maoHumano.value);
  const resAI = avaliarMaoAjiley(maoAI.value);

  const idxMao = maoAtual.value - 1;
  if (historicoHumano.value.length <= idxMao) {
    historicoHumano.value.push(0);
    historicoIA.value.push(0);
  }
  historicoHumano.value[idxMao] = resHumano.pontos;
  historicoIA.value[idxMao] = resAI.pontos;

  let resultado = `Tu mano: <b>${resHumano.pontos} pts</b> (${resHumano.naipe})<br>IA: <b>${resAI.pontos} pts</b> (${resAI.naipe})<br>`;

  if (resHumano.pontos >= resAI.pontos) {
    resultado += `🎉 <b>¡Ganaste la mano ${maoAtual.value}!</b>`;
  } else {
    resultado += `🤖 <b>La IA ganó la mano ${maoAtual.value}.</b>`;
  }

  mensagemResultado.value = resultado;

  if (
    maoAtual.value >= 3 &&
    totalHumano.value === totalIA.value &&
    historicoHumano.value.length === 3
  ) {
    faseJogo.value = 'SHOWDOWN';
    mensagemStatus.value =
      '¡Empate exacto de puntos en la grade! Se jugará una mano adicional de desempate.';
  } else if (maoAtual.value >= 3 && totalHumano.value !== totalIA.value) {
    verificarFimPartida();
  } else if (maoAtual.value >= 4) {
    verificarFimPartida();
  } else {
    faseJogo.value = 'SHOWDOWN';
    mensagemStatus.value = `Mano ${maoAtual.value} finalizada. Las fichas permanecen en el pozo. Prepara la siguiente.`;
  }
};

const verificarFimPartida = () => {
  faseJogo.value = 'FIN_PARTIDA';
  let finalMsg = `Puntuación total (Grade) — Tú: <b>${totalHumano.value} pts</b> | IA: <b>${totalIA.value} pts</b><br><br>`;

  const premioPozoTotal = pote.value;

  if (totalHumano.value > totalIA.value) {
    fichasHumano.value += premioPozoTotal;
    pote.value = 0;
    finalMsg += `👑 <span class="text-amber-3 text-weight-bold">¡TRIUNFO ÉPICO! Conquistaste la victoria y barriste con todo el pozo acumulado (${premioPozoTotal} fichas). ¡Eres el gran Maestro del Ajiley!</span>`;
    triggerWinEffects();
  } else if (totalHumano.value < totalIA.value) {
    fichasIA.value += premioPozoTotal;
    pote.value = 0;
    finalMsg += `🤖 <b>La IA ganó la partida y se lleva todo el pozo acumulado (${premioPozoTotal} fichas).</b>`;
  } else {
    const mitad = Math.floor(premioPozoTotal / 2);
    fichasHumano.value += mitad;
    fichasIA.value += mitad;
    pote.value = 0;
    finalMsg += `🤝 <b>¡Empate absoluto! Se reparte el pozo acumulado (${premioPozoTotal} fichas).</b>`;
    triggerWinEffects();
  }

  mensagemResultadoFinal.value = finalMsg;
  mensagemStatus.value = 'Partida finalizada. Pozo entregado al ganador.';
};

const exitGame = () => {
  router.push('/');
};

onMounted(() => {
  reiniciarPartidaCompleta();
});
</script>

<style scoped>
.ajiley-page {
  background: radial-gradient(circle at center, #0e2a6b 0%, #061336 100%);
  min-height: 100vh;
  width: 100%;
}

.game-container {
  background: rgba(15, 35, 95, 0.85);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.6);
  width: 95vw !important;
  margin: 0 auto;
}

.status-banner {
  width: 100%;
  background: rgba(0, 0, 0, 0.35);
  border-left: 3px solid #ffb300;
  border-radius: 0 6px 6px 0;
  font-weight: 400;
  font-size: 12px;
}

.game-table-felt {
  width: 100%;
  background: radial-gradient(circle, #105c47 0%, #063024 100%);
  border: 3px solid rgba(212, 175, 55, 0.35);
  border-radius: 14px;
  box-shadow:
    inset 0 4px 15px rgba(0, 0, 0, 0.8),
    0 6px 18px rgba(0, 0, 0, 0.4);
}

.score-grid-box {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(0, 0, 0, 0.75);
  border: 1px solid rgba(212, 175, 55, 0.5);
  border-radius: 6px;
  padding: 4px 6px;
  z-index: 10;
}

.table-pot-display-top {
  position: absolute;
  top: 10px;
  right: 10px;
  background: rgba(0, 0, 0, 0.75);
  border: 1px solid rgba(255, 193, 7, 0.6);
  border-radius: 6px;
  z-index: 10;
}

.ajiley-alert-box {
  background: rgba(255, 193, 7, 0.2);
  border: 2px solid #ffd700;
  border-radius: 10px;
  width: 100%;
  max-width: 450px;
  animation: ajileyPulse 1s infinite alternate;
}

@keyframes ajileyPulse {
  0% {
    box-shadow: 0 0 10px rgba(255, 215, 0, 0.4);
  }
  100% {
    box-shadow: 0 0 25px rgba(255, 215, 0, 0.8);
  }
}

.table-card-back-compact {
  width: 58px;
  height: 78px;
  background: linear-gradient(135deg, #1565c0 0%, #0d47a1 100%);
  border-radius: 8px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.4);
}

.player-card-compact {
  position: relative;
  width: 64px;
  height: 86px;
  background-color: #ffffff;
  border-radius: 8px;
  cursor: pointer;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
  border: 2px solid transparent;
  transition:
    transform 0.2s ease,
    border-color 0.2s ease;
}

.player-card-compact:hover:not(.disabled-card) {
  transform: translateY(-6px) scale(1.02);
}

.selected-card {
  border-color: #ffc107 !important;
  transform: translateY(-8px);
  box-shadow: 0 0 10px rgba(255, 193, 7, 0.8);
}

.disabled-card {
  cursor: default;
}

.atlas-img {
  position: absolute;
  top: 0;
  left: 0;
  width: 1000%;
  height: 400%;
  max-width: none;
  pointer-events: none;
}

.game-chip {
  background: rgba(0, 0, 0, 0.4) !important;
  color: #fff !important;
  border: 1px solid rgba(255, 255, 255, 0.15);
  font-weight: 400;
  font-size: 11px;
}

.action-btn {
  width: 105px;
  font-size: 10px;
  border-radius: 6px;
}

.results-table-banner {
  background: rgba(0, 0, 0, 0.92);
  border: 2px solid rgba(212, 175, 55, 0.8);
  border-radius: 12px;
  width: 100%;
  max-width: 400px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.9);
}

.win-glow-box {
  animation: winGlowPulse 1.5s infinite alternate;
}

/* EFECTO ESPECIAL DE TRIUNFO ÉPICO PARA EL HUMANO */
.win-glory-box {
  animation: winGloryPulse 1.2s infinite alternate;
  background: radial-gradient(circle, rgba(20, 60, 40, 0.95) 0%, rgba(5, 20, 15, 0.95) 100%);
  border: 3px solid #00ffcc !important;
}

@keyframes winGloryPulse {
  0% {
    box-shadow:
      0 0 20px rgba(0, 255, 204, 0.6),
      inset 0 0 15px rgba(0, 255, 204, 0.3);
    border-color: rgba(0, 255, 204, 0.8);
  }
  100% {
    box-shadow:
      0 0 45px rgba(0, 255, 204, 1),
      inset 0 0 25px rgba(0, 255, 204, 0.7);
    border-color: #00ffcc;
  }
}

@keyframes winGlowPulse {
  0% {
    box-shadow:
      0 0 15px rgba(255, 215, 0, 0.6),
      inset 0 0 10px rgba(255, 215, 0, 0.3);
    border-color: rgba(255, 215, 0, 0.8);
  }
  100% {
    box-shadow:
      0 0 35px rgba(255, 215, 0, 0.9),
      inset 0 0 20px rgba(255, 215, 0, 0.6);
    border-color: #ffd700;
  }
}

.score-table {
  font-size: 10px;
  border-collapse: collapse;
  color: #fff;
}

.score-table th,
.score-table td {
  padding: 1px 4px;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.dialog-effect {
  background: rgba(13, 27, 75, 0.95) !important;
  backdrop-filter: blur(12px);
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.15);
}

@media (max-width: 599px) {
  .game-container {
    padding: 8px !important;
    padding-top: 45px !important;
  }

  .score-grid-box {
    top: 6px;
    left: 6px;
    transform: scale(0.9);
    transform-origin: top left;
  }

  .table-pot-display-top {
    top: 6px;
    right: 6px;
    transform: scale(0.9);
    transform-origin: top right;
  }

  .top-header-section {
    margin-top: 15px;
  }

  .player-card-compact {
    width: 54px;
    height: 74px;
  }

  .table-card-back-compact {
    width: 48px;
    height: 64px;
  }

  .action-buttons-group {
    gap: 4px;
  }

  .action-buttons-group .q-btn {
    font-size: 9px !important;
    padding: 4px 8px !important;
  }
}
</style>
