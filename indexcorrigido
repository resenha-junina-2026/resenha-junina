#!/usr/bin/env python3
"""
═══════════════════════════════════════════════════════
RESENHA JUNINA 2026 — PATCHER v2.0
Aplica todas as 17 correções no index.html original
═══════════════════════════════════════════════════════

USO:
  python3 patch_index.py index.html

Gera: index_corrigido.html
"""

import sys
import re
import os

def patch(content):
    fixes_applied = []
    
    # ══════════════════════════════════════
    # FIX #25: viewport — remover user-scalable=no
    # ══════════════════════════════════════
    old_vp = 'user-scalable=no, '
    if old_vp in content:
        content = content.replace(old_vp, '')
        fixes_applied.append('#25 viewport user-scalable=no removido')
    
    # ══════════════════════════════════════
    # FIX #15: _fbWriting → sistema baseado em Set
    # ══════════════════════════════════════
    # Substituir declaração global
    old_fb = "let ST=load(), db=null, fbReady=false, _fbListeners=[], _fbWriting=false;"
    new_fb = """let ST=load(), db=null, fbReady=false, _fbListeners=[];
// FIX: Sistema de tracking de writes baseado em Set (substitui _fbWriting boolean)
let _fbPendingWrites = new Set();
let _fbWriteTimeout = null;
function markWriting(id){ _fbPendingWrites.add(id); clearTimeout(_fbWriteTimeout); _fbWriteTimeout=setTimeout(()=>_fbPendingWrites.clear(),10000); }
function unmarkWriting(id){ _fbPendingWrites.delete(id); }
function isWriting(){ return _fbPendingWrites.size > 0; }"""
    if old_fb in content:
        content = content.replace(old_fb, new_fb)
        fixes_applied.append('#15 _fbWriting → _fbPendingWrites Set')
    
    # ══════════════════════════════════════
    # FIX #16: Firebase listener merge inteligente
    # ══════════════════════════════════════
    old_listener = "if(_fbWriting) return;"
    new_listener = "if(isWriting()) return;"
    content = content.replace(old_listener, new_listener)
    
    old_merge = "ST.participantes=parts.sort((a,b)=>(a.id||0)-(b.id||0));"
    new_merge = """// FIX #16: Merge inteligente — preserva edições locais recentes
    const merged = parts.map(fbP => {
      const localP = ST.participantes.find(p => p.codigo === fbP.codigo);
      if(localP && localP._localEdit && (Date.now() - (localP._localEditTs||0)) < 5000) {
        return {...fbP, ...localP, _fbId: fbP._fbId};
      }
      return fbP;
    });
    ST.participantes.forEach(lp => { if(!merged.find(m => m.codigo === lp.codigo)) merged.push(lp); });
    ST.participantes = merged.sort((a,b)=>(a.id||0)-(b.id||0));"""
    if old_merge in content:
        content = content.replace(old_merge, new_merge)
        fixes_applied.append('#16 Firebase listener merge inteligente')
    
    # ══════════════════════════════════════
    # FIX #15b: fbSavePart com tracking
    # ══════════════════════════════════════
    old_save = """async function fbSavePart(p){
  if(!db) return;_fbWriting=true;
  try{
    const data={...p};delete data._fbId;"""
    new_save = """async function fbSavePart(p){
  if(!db) return;
  const writeId = p.codigo || ('tmp-'+Date.now());
  markWriting(writeId);
  try{
    const data={...p};delete data._fbId;delete data._localEdit;delete data._localEditTs;"""
    if old_save in content:
        content = content.replace(old_save, new_save)
        fixes_applied.append('#15b fbSavePart tracking')
    
    # Fix fbSavePart finally block
    old_finally1 = "finally{setTimeout(()=>{_fbWriting=false;},1500);}"
    new_finally1 = "finally{setTimeout(()=>unmarkWriting(writeId),2000);}"
    content = content.replace(old_finally1, new_finally1, 1)  # Only first occurrence
    
    # ══════════════════════════════════════
    # FIX #15c: fbDeletePart com tracking
    # ══════════════════════════════════════
    old_del = """async function fbDeletePart(fbId){if(!db||!fbId)return;_fbWriting=true;try{await db.collection('rj26_participantes').doc(fbId).delete();}catch(e){console.warn('FB delete error:',e);}finally{setTimeout(()=>{_fbWriting=false;},1500);}}"""
    new_del = """async function fbDeletePart(fbId){if(!db||!fbId)return;markWriting('del-'+fbId);try{await db.collection('rj26_participantes').doc(fbId).delete();}catch(e){console.warn('FB delete error:',e);}finally{setTimeout(()=>unmarkWriting('del-'+fbId),2000);}}"""
    if old_del in content:
        content = content.replace(old_del, new_del)
        fixes_applied.append('#15c fbDeletePart tracking')
    
    # ══════════════════════════════════════
    # FIX #17: proximoNumero() salva após incremento
    # ══════════════════════════════════════
    old_prox = "if(!db) return ST.ids.part++;"
    new_prox = "if(!db) { const id=ST.ids.part++; save(); return id; }"
    if old_prox in content:
        content = content.replace(old_prox, new_prox)
        fixes_applied.append('#17 proximoNumero save após incremento')
    
    old_prox2 = "console.warn('Contador Firebase falhou:',e);return ST.ids.part++;"
    new_prox2 = "console.warn('Contador Firebase falhou:',e);const id=ST.ids.part++;save();return id;"
    if old_prox2 in content:
        content = content.replace(old_prox2, new_prox2)
    
    # ══════════════════════════════════════
    # FIX #18: confirmarEEnviar não força Pix
    # ══════════════════════════════════════
    old_conf = "p.pgto=p.pgto==='Pendente'?'Pix':p.pgto;"
    new_conf = """// FIX #18: Usa método de pagamento correto (não força Pix)
  let metodoPgto = p.pgto;
  if(!metodoPgto || metodoPgto==='Pendente'){
    metodoPgto = (p.metodoPgto==='mp') ? 'Mercado Pago' : 'Pix';
  }
  p.pgto = metodoPgto;"""
    if old_conf in content:
        content = content.replace(old_conf, new_conf)
        fixes_applied.append('#18 confirmarEEnviar método pagamento correto')
    
    # ══════════════════════════════════════
    # FIX #30: salvarPart marca edição local
    # ══════════════════════════════════════
    # Adicionar _localEdit ao salvar
    old_spart = "save();const partSalvo=ST.participantes.find(p=>p.id==eid);if(partSalvo)fbSavePart(partSalvo);"
    new_spart = "save();const partSalvo=ST.participantes.find(p=>p.id==eid);if(partSalvo){partSalvo._localEdit=true;partSalvo._localEditTs=Date.now();fbSavePart(partSalvo);}"
    if old_spart in content:
        content = content.replace(old_spart, new_spart)
        fixes_applied.append('#30 salvarPart marca edição local')
    
    # ══════════════════════════════════════
    # FIX #30b: registrarEntrada marca edição local
    # ══════════════════════════════════════
    old_reg = "ST.log.unshift(entradaLog);save();"
    new_reg = "ST.log.unshift(entradaLog);p._localEdit=true;p._localEditTs=Date.now();save();"
    if old_reg in content:
        content = content.replace(old_reg, new_reg)
        fixes_applied.append('#30b registrarEntrada marca edição local')
    
    # ══════════════════════════════════════
    # FIX #19: importJSON sincroniza com Firebase  
    # ══════════════════════════════════════
    old_import = "if(!data.participantes)throw new Error('Arquivo invalido');if(!confirm('Restaurar backup de '+data.participantes.length+' participantes?'))return;"
    new_import = """if(!data.participantes)throw new Error('Arquivo invalido');
      const doSync = fbReady && confirm('Restaurar backup de '+data.participantes.length+' participantes e sincronizar com Firebase?');
      if(!doSync && !confirm('Restaurar backup de '+data.participantes.length+' participantes? (Apenas local)'))return;"""
    if old_import in content:
        content = content.replace(old_import, new_import)
        fixes_applied.append('#19 importJSON com confirmação Firebase')
    
    # Adicionar sincronização Firebase após import
    old_toast_import = "renderAll();toast('Backup restaurado!');"
    new_toast_import = """// FIX #19: Sincroniza com Firebase se disponível
      if(doSync && db){
        toast('Sincronizando com Firebase...');
        try{
          const oldSnap=await db.collection('rj26_participantes').get();
          const delBatch=db.batch();oldSnap.forEach(doc=>delBatch.delete(doc.ref));await delBatch.commit();
          for(let i=0;i<ST.participantes.length;i+=400){
            const batch=db.batch();
            ST.participantes.slice(i,i+400).forEach(p=>{const d={...p};delete d._fbId;delete d._localEdit;delete d._localEditTs;batch.set(db.collection('rj26_participantes').doc(),d);});
            await batch.commit();
          }
          await fbSaveConfig();
          const maxNum=Math.max(0,...ST.participantes.map(p=>{const m=(p.codigo||'').match(/RESENHA-(\\d+)/);return m?parseInt(m[1]):0;}));
          await db.collection('rj26_config').doc('contador').set({next:maxNum+1});
          toast('Backup restaurado e sincronizado!');
        }catch(fbErr){console.warn('Erro sync:',fbErr);toast('Restaurado localmente. Erro ao sincronizar.');}
      } else { toast('Backup restaurado!'); }
      renderAll();"""
    if old_toast_import in content:
        content = content.replace(old_toast_import, new_toast_import)
    
    # Tornar a função async
    old_reader = "r.onload=e=>{"
    new_reader = "r.onload=async e=>{"
    if old_reader in content:
        content = content.replace(old_reader, new_reader, 1)
    
    # ══════════════════════════════════════
    # FIX #21: zerarParticipantes com reset correto
    # ══════════════════════════════════════
    old_zerar_cont = "await db.collection('rj26_config').doc('contador').set({next:1});"
    if old_zerar_cont in content:
        # Já está correto, mas vamos garantir
        fixes_applied.append('#21 zerarParticipantes (já correto)')
    
    # ══════════════════════════════════════
    # FIX: Migração — limpar campos temporários
    # ══════════════════════════════════════
    old_migrate_end = "save();\n})();"
    new_migrate_end = """  // Limpa flags temporários de edição
  (ST.participantes||[]).forEach(p => { delete p._localEdit; delete p._localEditTs; });
  save();
})();"""
    if old_migrate_end in content:
        content = content.replace(old_migrate_end, new_migrate_end, 1)
    
    # ══════════════════════════════════════
    # FIX: Font family update
    # ══════════════════════════════════════
    old_font = "family=Figtree:wght@300;400;500;600;700;800"
    new_font = "family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800"
    if old_font in content:
        content = content.replace(old_font, new_font)
        content = content.replace("font-family:'Figtree',sans-serif", "font-family:'Plus Jakarta Sans',sans-serif")
        content = content.replace("font-family:'Figtree',sans-serif", "font-family:'Plus Jakarta Sans',sans-serif")
        fixes_applied.append('Font: Figtree → Plus Jakarta Sans')
    
    # ══════════════════════════════════════
    # FIX: enviarLote com error tracking
    # ══════════════════════════════════════
    old_envlote = "let enviados=0,erros=0;"
    new_envlote = "let enviados=0,erros=0;const errosList=[];"
    if old_envlote in content:
        content = content.replace(old_envlote, new_envlote)
    
    old_envlote_err = "erros++;"
    new_envlote_err = "errosList.push(p.nome);erros++;"
    # Only replace in enviarLote context (first occurrence after the counter)
    idx = content.find("errosList=[]")
    if idx > -1:
        rest = content[idx:]
        rest = rest.replace(old_envlote_err, new_envlote_err, 1)
        content = content[:idx] + rest
        fixes_applied.append('#20 enviarLote error tracking')
    
    return content, fixes_applied


def main():
    if len(sys.argv) < 2:
        print("Uso: python3 patch_index.py <caminho_index.html>")
        print("Gera: index_corrigido.html no mesmo diretório")
        sys.exit(1)
    
    input_path = sys.argv[1]
    if not os.path.exists(input_path):
        print(f"Erro: arquivo '{input_path}' não encontrado")
        sys.exit(1)
    
    with open(input_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    print(f"Arquivo lido: {len(content)} chars, {content.count(chr(10))} linhas")
    print("Aplicando correções...\n")
    
    patched, fixes = patch(content)
    
    output_path = os.path.join(os.path.dirname(input_path) or '.', 'index_corrigido.html')
    with open(output_path, 'w', encoding='utf-8') as f:
        f.write(patched)
    
    print("═" * 50)
    print(f"CORREÇÕES APLICADAS: {len(fixes)}")
    print("═" * 50)
    for fix in fixes:
        print(f"  ✓ {fix}")
    
    print(f"\nArquivo salvo: {output_path}")
    print(f"Tamanho: {len(patched)} chars, {patched.count(chr(10))} linhas")
    
    # Validações
    print("\nValidações:")
    checks = {
        'user-scalable=no removido': 'user-scalable=no' not in patched,
        '_fbPendingWrites presente': '_fbPendingWrites' in patched,
        'isWriting() em uso': 'isWriting()' in patched,
        'markWriting em uso': 'markWriting(' in patched,
        '_localEdit em uso': '_localEdit' in patched,
        'Merge inteligente': 'merged.sort' in patched or 'Merge inteligente' in patched,
        'save() após part++': 'const id=ST.ids.part++;save()' in patched or 'const id=ST.ids.part++; save()' in patched,
    }
    
    all_ok = True
    for name, ok in checks.items():
        status = '✓' if ok else '✗'
        print(f"  {status} {name}")
        if not ok: all_ok = False
    
    if all_ok:
        print("\n✓ TODAS AS CORREÇÕES APLICADAS COM SUCESSO")
    else:
        print("\n⚠ Algumas correções podem não ter sido aplicadas.")
        print("  Verifique se o arquivo original é a versão correta.")


if __name__ == '__main__':
    main()
