<script setup lang="ts">
type Simulation = 'normal' | 'fallo' | 'particion' | 'recuperado'

defineProps<{ simulation: Simulation; logs: string[] }>()
defineEmits<{ simulate: [state: Simulation] }>()
</script>

<template>
  <section class="simulator">
    <div class="section-heading">
      <div><span class="eyebrow">SYSTEM OBSERVABILITY SIMULATOR</span><h2>Provoca el fallo. Observa la coordinación.</h2></div>
      <span class="status" :class="simulation">{{ simulation === 'normal' ? 'CLUSTER: ESTABLE' : simulation === 'recuperado' ? 'CLUSTER: RECUPERADO' : 'CLUSTER: DEGRADADO' }}</span>
    </div>
    <div class="sim-controls">
      <button @click="$emit('simulate', 'fallo')">Simular fallo maestro</button>
      <button @click="$emit('simulate', 'particion')">Partición de red</button>
      <button @click="$emit('simulate', 'recuperado')">Recuperar</button>
      <button @click="$emit('simulate', 'normal')">Restablecer</button>
    </div>
    <div class="sim-grid">
      <div class="network" :class="simulation">
        <div class="client node"><b>CLIENTES</b><span>Solicitudes</span></div>
        <div class="network-line line-a"></div><div class="network-line line-b"></div>
        <div class="masters">
          <div class="node master" :class="{ failed: simulation === 'fallo' || simulation === 'particion' }"><span class="pulse"></span><b>NODO 01</b><small>{{ simulation === 'fallo' ? 'FALLO' : simulation === 'particion' ? 'AISLADO' : 'MAESTRO' }}</small></div>
          <div class="node standby" :class="{ promoted: simulation !== 'normal' }"><b>NODO 02</b><small>{{ simulation === 'normal' ? 'EN ESPERA' : 'NUEVO MAESTRO' }}</small></div>
        </div>
        <div class="replicas"><div v-for="n in 3" :key="n" class="node replica"><span>▤</span><small>RÉPLICA {{ n }}</small></div></div>
        <span class="network-label">BLOQUEO EXCLUSIVO + HEARTBEAT + QUÓRUM</span>
      </div>
      <div class="terminal">
        <div class="terminal-head"><span>●</span> SYSTEM_LOG_OUTPUT</div>
        <p v-for="(log, i) in logs" :key="log"><time>[12:05:0{{ i + 1 }}]</time> {{ log }}</p>
        <div class="terminal-question"><span>TRAMPA DE EXAMEN</span>¿Por qué un timeout no basta para evitar dos maestros?</div>
      </div>
    </div>
    <div class="simulation-explain">
      <strong>{{ simulation === 'normal' ? 'Estado normal' : simulation === 'fallo' ? 'Conmutación por error' : simulation === 'particion' ? 'Partición y split-brain' : 'Reincorporación segura' }}</strong>
      <p v-if="simulation === 'normal'">El maestro activo conserva el bloqueo exclusivo y emite heartbeats. El nodo en espera replica estado, pero no tiene autoridad para escribir.</p>
      <p v-else-if="simulation === 'fallo'">Al expirar el lease, se libera la autoridad anterior. El nodo en espera obtiene mayoría, adquiere el bloqueo y continúa el servicio.</p>
      <p v-else-if="simulation === 'particion'">La rama con mayoría elige líder. La rama minoritaria debe rechazar escrituras; los fencing tokens impiden que un maestro aislado opere con autoridad obsoleta.</p>
      <p v-else>El nodo recuperado no vuelve como maestro: primero sincroniza el registro y se reincorpora como seguidor para evitar sobrescribir decisiones confirmadas.</p>
    </div>
  </section>
</template>
