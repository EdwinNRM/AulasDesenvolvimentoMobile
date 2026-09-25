import { Pressable, StyleSheet, Text, View } from 'react-native';
import type { Tarefa } from '../types/Tarefa';

type Props = {
  tarefa: Tarefa;
  onAlternar: () => void;
  onExcluir: () => void;
};

export function TaskItem({ tarefa, onAlternar, onExcluir }: Props) {
  return (
    <View style={styles.item}>
      <Pressable style={styles.conteudo} onPress={onAlternar}
        accessibilityRole="checkbox"
        accessibilityState={{ checked: tarefa.concluida }}
        accessibilityLabel={tarefa.titulo}>
        <View style={[styles.circulo, tarefa.concluida && styles.circuloMarcado]}>
          {tarefa.concluida ? <Text style={styles.marca}>✓</Text> : null}
        </View>
        <Text style={styles.titulo}>{tarefa.titulo}</Text>
      </Pressable>
      <Pressable onPress={onExcluir} accessibilityRole="button"
        accessibilityLabel={`Excluir ${tarefa.titulo}`}>
        <Text style={styles.excluir}>Excluir</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  item: { flexDirection: 'row', alignItems: 'center', backgroundColor: '#FFFFFF',
    borderRadius: 14, paddingHorizontal: 16, paddingVertical: 16, marginBottom: 10,
    borderWidth: 1, borderColor: '#E5EAF2' },
  conteudo: { flex: 1, flexDirection: 'row', alignItems: 'center', gap: 12 },
  circulo: { width: 24, height: 24, borderRadius: 12, borderWidth: 2,
    borderColor: '#2563EB', alignItems: 'center', justifyContent: 'center' },
  circuloMarcado: { backgroundColor: '#2563EB' },
  marca: { color: '#FFFFFF', fontSize: 14, fontWeight: '700' },
  titulo: { flex: 1, color: '#1E293B', fontSize: 16 },
  excluir: { color: '#DC2626', fontSize: 13, fontWeight: '600', paddingLeft: 10 },
});
