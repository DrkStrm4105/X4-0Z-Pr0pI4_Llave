# Exploración 0001 — Intel ME, metakernel ternario y el lenguaje TRÍA

**Estado:** exploratorio. No normativo. No valida ninguna hipótesis criptográfica (invariante 10).
**Fecha:** 2026-10-30

## 0. Convención epistémica

Este documento mezcla deliberadamente análisis y sueño, a petición del propietario. Para que la mezcla no contamine la evidencia, cada afirmación relevante lleva una marca:

- **[H] Hecho:** documentado públicamente y verificable.
- **[I] Inferencia:** razonamiento propio a partir de hechos; discutible.
- **[S] Sueño:** especulación libre, imagen o metáfora. Valor heurístico, no probatorio.

Las cifras marcadas como *ilustrativas* son inventadas para mostrar la forma de un cálculo, no su resultado.

---

## Parte I — Economía del Intel Management Engine

### 1.1 Qué es

- **[H]** El Intel Management Engine (ME; en versiones recientes, CSME) es un subsistema autónomo integrado en el chipset (PCH) de los equipos Intel desde mediados de los 2000. Tiene procesador, memoria y firmware propios, acceso a la memoria del sistema y a la red, y funciona mientras el equipo tenga alimentación, incluso "apagado".
- **[H]** Desde ME 11 (generación Skylake, 2015) ejecuta MINIX 3 sobre un núcleo x86 derivado de Quark. Lo documentó públicamente Positive Technologies en 2017.
- **[H]** Sobre él se construye AMT (Active Management Technology), la administración remota fuera de banda de la gama empresarial vPro. El ME existe también en equipos sin vPro, con funciones reducidas: arranque, inicialización, DRM y TPM por firmware (PTT).
- **[H]** En 2017 se publicaron fallos graves: CVE-2017-5689 (INTEL-SA-00075), que permitía saltarse la autenticación de AMT, y el boletín INTEL-SA-00086, con vulnerabilidades en ME, TXE y SPS.
- **[H]** Positive Technologies halló el bit *HAP* (High Assurance Platform), un modo que deja el ME casi inerte tras el arranque. Se asocia a requisitos de clientes gubernamentales de EE. UU. Herramientas como `me_cleaner` lo explotan, y algunos fabricantes (Purism, System76) venden equipos con el ME neutralizado.
- **[H]** Hay análogos en toda la industria: AMD PSP, ARM TrustZone, Apple Secure Enclave/T2, Google Titan, Microsoft Pluton (anunciado en 2020). Como contrapunto abierto existen OpenTitan (2019) y Caliptra (2022, impulsado por AMD, Google, Microsoft y NVIDIA).

### 1.2 Modelo de negocio: un mismo silicio, varios precios

- **[I] Discriminación de precios por versionado.** El hardware de gestión se fabrica en casi todos los chipsets, y la función premium (AMT/vPro) se habilita o se fusiona según la gama. Es la estrategia clásica de "dañar" un producto para segmentar (Shapiro y Varian, *Information Rules*). El coste marginal de incluir el ME en cada chip es bajo. El margen se captura en la gama vPro.
- **[I] Economías de alcance.** Un único subsistema sirve a la vez para gestión remota, arranque seguro, DRM y TPM. Separarlos costaría más silicio y más validación, así que el incentivo empuja a concentrar funciones en un solo componente privilegiado.
- **[I] Dependencia de la trayectoria.** Cuando el arranque y la inicialización dependen del ME, quitarlo deja de ser opcional. La decisión de 2006 se convierte en un coste hundido arquitectónico que condiciona cada generación posterior.

### 1.3 Fallos de mercado

| Mecanismo | Cómo se manifiesta en el ME | Marca |
|---|---|---|
| **Bien de confianza** (Darby y Karni, 1973) | El comprador no puede verificar la calidad ni la inocuidad del firmware, ni antes ni después de comprar. | [I] |
| **Asimetría de información** (Akerlof, 1970) | Intel conoce el código; el usuario solo ve una caja negra firmada. Sin señal creíble, el mercado no premia el firmware seguro frente al que no lo es. | [I] |
| **Externalidad negativa** | La gama empresarial obtiene el beneficio (AMT). El riesgo residual lo soportan todos los compradores, también quienes nunca usarán AMT. | [I] |
| **Riesgo sistémico** | Un componente homogéneo en cientos de millones de equipos convierte un fallo en un evento correlacionado, no en riesgos independientes. | [I] |
| **Poder de mercado** | Con dos grandes proveedores x86 que siguen la misma lógica (ME y PSP), el usuario no tiene una opción de compra real sin subsistema privilegiado. | [I] |

### 1.4 Lectura contable

- **[I] Pasivo contingente** (NIC 37 / IAS 37). Cada chip vendido lleva una obligación posible: parches, soporte, litigios y reputación, que se materializa al descubrirse una vulnerabilidad. Mientras la probabilidad no sea "más probable que no", no se provisiona, solo se revela, si acaso. Así el riesgo queda fuera del balance hasta que estalla.
- **[I] Traslado de costes.** El parche de firmware lo publica Intel, pero lo integra y distribuye cada fabricante (OEM) en su BIOS, y lo instala el usuario. El coste de remediación se reparte aguas abajo y no aparece en la cuenta de resultados de quien diseñó el riesgo.
- **[I] Intangible de doble filo.** Para el segmento empresarial, la capacidad de gestión es un activo. Para el resto es un pasivo latente. Vender el mismo componente a ambos sin separar las cuentas es mezclar activo y pasivo en una sola línea.

**Esquema de cálculo (cifras ilustrativas):**

```text
Pérdida esperada agregada  E[L] = N · p · c
  N = unidades expuestas             (ilustrativo: 10⁸)
  p = prob. de explotación por unidad y año    (ilustrativo: 10⁻⁴)
  c = coste medio por incidente      (ilustrativo: 2 000 €)
  E[L] = 10⁸ · 10⁻⁴ · 2 000 = 2·10⁷ €/año

Pero con riesgo correlacionado (un fallo afecta a todos a la vez),
la varianza no se diluye con N: la cola es la que importa.
```

**[I]** La segunda línea es la lección central: en riesgos correlacionados, la media engaña. Una aseguradora no diversifica un fallo que afecta a toda la flota a la vez.

### 1.5 Teoría de juegos

- **[I] Jugadores:** Intel, fabricantes (OEM), empresas que compran vPro, usuarios particulares y Estados.
- **[I]** El bit HAP muestra un **equilibrio con trato diferenciado**: el actor con poder de negociación (un Estado) obtiene la opción de salida; el usuario común, no. No es un fallo técnico, es una asignación de poder.
- **[I]** Ningún OEM gana por sí solo desactivando el ME: asume costes de validación y pierde certificaciones. Es un dilema colectivo, que solo se rompe con coordinación (Caliptra, OpenTitan) o con regulación.

### 1.6 Auditoría frente a los invariantes de este repositorio

| Invariante | ¿Lo respeta el ME por defecto? | Comentario |
|---|---|---|
| 1. Soberanía | ✗ | El sujeto no puede negar ni revocar de forma efectiva. |
| 2. Separación | ✗ | Gestión, arranque, DRM y TPM conviven en un único componente. |
| 3. Minimización | ✗ | Expone capacidad de red y de acceso a memoria aunque no se use. |
| 4. Temporalidad | ~ | Hay versiones y parches, pero sin vigencia explícita de privilegios. |
| 5. Trazabilidad | ✗ | El usuario no puede reconstruir qué hizo el subsistema. |
| 6. Revocabilidad | ~ | AMT se puede desactivar; el ME en sí, no oficialmente. |
| 7. Recuperabilidad | ~ | Un fallo de firmware puede inutilizar el equipo. |
| 8. No amplificación | ✗ | Es, por construcción, un componente de privilegio superior al sistema operativo. |
| 9. Interoperabilidad | — | No aplica directamente. |
| 10. Falsabilidad | ✗ | Código cerrado: no admite revisión independiente plena. |

**[I] Diagnóstico.** El ME codifica el permiso como `+1` por defecto. Tu especificación prohíbe exactamente esa traducción: *"La ausencia de decisión permanece en `0`; nunca se traduce automáticamente a `+1`."* El ME es el contraejemplo canónico de tu estado ternario.

### 1.7 Diseño alternativo (la gestión que habría hecho alguien de ADE)

1. **[I] Activación por solicitud del usuario:** el subsistema nace en `0` y pasa a `+1` solo con una autorización explícita, contextual y con vigencia.
2. **[I] Separar las cuentas:** gestión remota como producto aparte, con su propio silicio o activación, y su propio pasivo contingente reconocido.
3. **[I] Señal creíble:** código abierto o auditoría independiente publicada, para que el mercado pueda pagar por seguridad verificable.
4. **[I] Interruptor físico de salida:** una opción de salida real y no negociada, como el HAP, pero para todos.
5. **[H/I]** Caliptra y OpenTitan apuntan en esa dirección: una raíz de confianza abierta y auditable.

---

## Parte II — Metakernel: fundamentos

### 2.1 Linaje

- **[H]** Las **capacidades** (Dennis y Van Horn, 1966): una referencia que es a la vez nombre y permiso. Las desarrollaron KeyKOS, EROS, Capsicum y el lenguaje E.
- **[H]** El **núcleo de separación** (Rushby, 1981): aislar particiones como si fueran máquinas distintas.
- **[H]** El **microkernel** (familia L4). seL4 fue verificado formalmente en 2009: existe una prueba matemática de que el código cumple su especificación.
- **[H]** El **exokernel** (MIT, 1995): el núcleo solo multiplexa recursos y la política se deja a las bibliotecas.
- **[I]** La jerarquía informal de "anillos": ring 3 (usuario), ring 0 (núcleo), ring −1 (hipervisor), ring −2 (SMM) y "ring −3" (ME/PSP). Cada nivel inferior ve y controla los superiores, nunca al revés.

### 2.2 Definición propuesta

> **[I] Metakernel:** núcleo cuya única función es mediar capacidades entre núcleos, sin ejecutar política propia ni poseer privilegio que no le haya sido delegado.

El ME es lo contrario: un "metakernel" de facto que posee privilegio sin haberlo recibido del sujeto. La propuesta consiste en invertir la flecha: que la autoridad fluya **desde el sujeto hacia abajo**, y no desde el fabricante hacia arriba.

### 2.3 Axiomas (derivados de tus invariantes)

Sea **T = {−1, 0, +1}** con el orden −1 < 0 < +1.

- **A1. Retículo:** `a ∧ b = min(a, b)`, `a ∨ b = max(a, b)` y `¬a = −a`. Es la lógica fuerte de Kleene (1938), y coincide con la aritmética ternaria equilibrada en su signo.
- **A2. Ausencia neutra:** `decidir(∅) = 0`. Ninguna regla puede producir `+1` sin una capacidad explícita.
- **A3. Atenuación monótona** (no amplificación, invariante 8): si `c' = atenuar(c)`, entonces `acciones(c') ⊆ acciones(c)` y `vigencia(c') ≤ vigencia(c)`.
- **A4. Revocación hereditaria:** revocar una capacidad lleva a `−1` toda su descendencia, sin borrar el pasado (invariante 6).
- **A5. Expiración neutra:** `Expirar: +1 × t → 0`. Caducar no es denegar: es volver a la indeterminación.
- **A6. Libro encadenado:** cada transición referencia el hash de la anterior (`previous_event_id`). Reescribir el pasado se vuelve detectable (invariante 5).

**[I] Propiedad clave.** Con A1 y A2, la conjunción de políticas nunca crea autorización: si cualquier condición es `0`, el resultado es como mucho `0`. El sistema falla hacia la indeterminación, no hacia el permiso.

### 2.4 Topologías

Respondiendo a tu intuición sobre árboles, cascadas y anillos:

| Topología | Uso en el metakernel | Marca |
|---|---|---|
| **Árbol** | Delegación: cada rama solo puede atenuar a su padre (A3). Revocar una rama poda su subárbol (A4). | [I] |
| **Cascada** | Canalización: cada etapa aplica `∧`, así que el permiso solo puede bajar a lo largo de la cadena. | [I] |
| **Anillo** | Auditoría mutua: cada núcleo verifica el libro del siguiente. Es la base natural de la recuperación por umbral (k de n fragmentos; Shamir, 1979) del operador `Recuperar`. | [I] |
| **Retículo** | El propio espacio de decisiones, donde se componen las políticas. | [I] |
| **Toro / red** [S] | Anillos de anillos: dispositivos que se auditan en un anillo, y anillos que se auditan entre sí. | [S] |

**[S] Circo Eterno y Reabsorción Fonámbula.** Leo tus términos como imágenes de este sistema. El *Circo Eterno* es el anillo de auditoría que nunca termina: cada núcleo observa al siguiente y nadie queda fuera del círculo. La *Reabsorción Fonámbula* es el destino de las capacidades caducadas: no se destruyen ni se niegan, caminan sobre la cuerda hacia el `0` y el sistema las reabsorbe. Son lecturas mías, no definiciones tuyas: corrígelas si no encajan.

---

## Parte III — Síntesis de lenguajes

### 3.1 Qué se toma de cada tradición

| Fuente | Aporte | Marca |
|---|---|---|
| Setun (Brusentsov, Universidad de Moscú, 1958) | Computación ternaria equilibrada real, en hardware. | [H] |
| Kleene / Łukasiewicz | Lógicas trivalentes formales. | [H] |
| SQL `NULL` | Advertencia: una 3VL mal comprendida produce errores sutiles. | [H/I] |
| E (Mark Miller) | Capacidades como objetos; sin autoridad ambiente. | [H] |
| Rust | Tipos afines: un valor se mueve, no se copia sin permiso. | [H] |
| Haskell | Efectos explícitos en el tipo. | [H] |
| Datalog (Biscuit, Zanzibar) | Políticas de autorización declarativas y decidibles. | [H] |
| Contabilidad por partida doble | Conservación: todo débito tiene su crédito. | [H/I] |

### 3.2 TRÍA: un lenguaje sintetizado

**[S/I]** TRÍA es un lenguaje pequeño de políticas y capacidades con estas reglas:

1. El tipo base es `trit` (`-1 | 0 | +1`). No existe un `bool` implícito.
2. Las capacidades son **lineales**: se usan, se delegan atenuadas o se revocan, pero no se duplican.
3. El único constructor de capacidad derivada es `atenuar`. No existe `ampliar`.
4. Toda traducción entre formatos debe declarar su pérdida (`pierde { campo, ... }`), o el compilador la rechaza (invariante 9).
5. Toda transición queda asentada en un libro encadenado. No hay efectos sin asiento.

**Gramática mínima (EBNF):**

```ebnf
politica  = "politica" ident "=" expr ";" ;
expr      = term { "|" term } ;
term      = factor { "&" factor } ;
factor    = "!" factor | "(" expr ")" | trit | ident ;
trit      = "-1" | "0" | "+1" ;
capacidad = "capacidad" ident "=" "emitir" "(" ident "," conjunto "," vigencia ")" ";"
          | "capacidad" ident "=" "atenuar" "(" ident "," conjunto "," vigencia ")" ";" ;
traduccion= "traducir" ident "->" ident "pierde" "{" [ ident { "," ident } ] "}" ";" ;
```

**Ejemplo [S]:**

```text
capacidad raiz = emitir(ana, {leer, firmar}, 10);
capacidad hija = atenuar(raiz, {leer}, 5);
politica arranque = dueño & firmware_auditado & !red_externa;
traducir PermisoV1 -> PermisoV2 pierde { nota_libre };
```

---

## Parte IV — Programas nuevos

### 4.1 Prototipo ejecutable

**[H]** El siguiente boceto en Python (sin dependencias) implementa los axiomas A1–A6 y un evaluador del subconjunto de políticas de TRÍA. Se ejecutó y su salida se reproduce debajo. **No tiene valor criptográfico:** el hash solo demuestra la forma del libro encadenado. Va embebido aquí, y no en `src/`, para respetar la regla del README de no introducir código antes de definir una cápsula mínima.

```python
"""TRÍA — prototipo de núcleo de permisos ternarios (experimental, sin valor criptográfico)."""
from dataclasses import dataclass, field
import hashlib, json, re

# --- Trit: lógica fuerte de Kleene sobre {-1, 0, +1} -------------------------
NEG, IND, POS = -1, 0, 1
def t_and(a, b): return min(a, b)          # conjunción = ínfimo
def t_or(a, b):  return max(a, b)          # disyunción = supremo
def t_not(a):    return -a                 # +1 <-> -1, 0 se queda en 0

# --- Registro de eventos encadenado (trazabilidad, invariante 5) ------------
@dataclass
class Libro:
    eventos: list = field(default_factory=list)
    def asentar(self, tipo, **datos):
        previo = self.eventos[-1]["id"] if self.eventos else None
        cuerpo = {"tipo": tipo, "previo": previo, **datos}
        cuerpo["id"] = hashlib.sha256(json.dumps(cuerpo, sort_keys=True).encode()).hexdigest()[:12]
        self.eventos.append(cuerpo)
        return cuerpo["id"]
    def verificar(self):
        previo = None
        for e in self.eventos:
            c = {k: v for k, v in e.items() if k != "id"}
            if c["previo"] != previo: return False
            if hashlib.sha256(json.dumps(c, sort_keys=True).encode()).hexdigest()[:12] != e["id"]: return False
            previo = e["id"]
        return True

# --- Capacidad: solo puede atenuarse, nunca amplificarse (invariante 8) -----
@dataclass(frozen=True)
class Capacidad:
    sujeto: str
    acciones: frozenset
    hasta: int            # vigencia (tiempo lógico)
    padre: object = None
    def atenuar(self, acciones, hasta):
        acciones = frozenset(acciones)
        if not acciones <= self.acciones or hasta > self.hasta:
            raise PermissionError("amplificación rechazada")
        return Capacidad(self.sujeto, acciones, hasta, self)

class Nucleo:
    """Metakernel mínimo: media toda decisión, la ausencia vale 0."""
    def __init__(self):
        self.libro, self.revocadas = Libro(), set()
    def emitir(self, sujeto, acciones, hasta):
        c = Capacidad(sujeto, frozenset(acciones), hasta)
        self.libro.asentar("emitir", sujeto=sujeto, acciones=sorted(acciones), hasta=hasta)
        return c
    def revocar(self, c):
        self.revocadas.add(id(c)); self.libro.asentar("revocar", sujeto=c.sujeto)
    def decidir(self, c, accion, ahora):
        if c is None: r = IND                              # sin decisión: 0, nunca +1
        else:
            linaje, x = [], c
            while x: linaje.append(x); x = x.padre
            if any(id(x) in self.revocadas for x in linaje): r = NEG
            elif ahora > c.hasta: r = IND                  # Expirar: +1 -> 0
            elif accion in c.acciones: r = POS
            else: r = IND
        self.libro.asentar("decidir", accion=accion, ahora=ahora, resultado=r)
        return r

# --- Mini-lenguaje de políticas: expresiones sobre trits --------------------
def evaluar(expr, entorno):
    """Gramática: e := e '&' e | e '|' e | '!' e | '(' e ')' | nombre | -1 | 0 | +1"""
    toks = re.findall(r"[()&|!]|[+-]?1|0|\w+", expr); pos = 0
    def ver(): return toks[pos] if pos < len(toks) else None
    def tomar():
        nonlocal pos; pos += 1; return toks[pos - 1]
    def o():
        v = y()
        while ver() == "|": tomar(); v = t_or(v, y())
        return v
    def y():
        v = u()
        while ver() == "&": tomar(); v = t_and(v, u())
        return v
    def u():
        t = tomar()
        if t == "!": return t_not(u())
        if t == "(":
            v = o(); tomar(); return v
        if t in ("-1", "0", "+1", "1"): return int(t)
        return entorno.get(t, IND)                         # variable desconocida = 0
    return o()

if __name__ == "__main__":
    n = Nucleo()
    raiz = n.emitir("ana", {"leer", "firmar"}, hasta=10)
    hija = raiz.atenuar({"leer"}, hasta=5)
    print("leer t=3   ->", n.decidir(hija, "leer", 3))
    print("firmar t=3 ->", n.decidir(hija, "firmar", 3))
    print("leer t=7   ->", n.decidir(hija, "leer", 7))
    print("sin capac. ->", n.decidir(None, "leer", 3))
    try: hija.atenuar({"leer", "firmar"}, 5)
    except PermissionError as e: print("ampliar    ->", e)
    n.revocar(raiz)
    print("tras revocar raíz, hija leer ->", n.decidir(hija, "leer", 3))
    print("libro íntegro:", n.libro.verificar(), "| eventos:", len(n.libro.eventos))
    n.libro.eventos[1]["resultado"] = NEG   # alguien reescribe el pasado
    print("libro tras manipulación:", n.libro.verificar())
    ent = {"dueño": POS, "firmware_auditado": IND, "red": NEG}
    print("politica 'dueño & firmware_auditado' ->", evaluar("dueño & firmware_auditado", ent))
    print("politica 'dueño & !red'              ->", evaluar("dueño & !red", ent))
    print("politica 'x_desconocida | 0'         ->", evaluar("x_desconocida | 0", ent))
```

**Salida real:**

```text
leer t=3   -> 1
firmar t=3 -> 0
leer t=7   -> 0
sin capac. -> 0
ampliar    -> amplificación rechazada
tras revocar raíz, hija leer -> -1
libro íntegro: True | eventos: 7
libro tras manipulación: False
politica 'dueño & firmware_auditado' -> 0
politica 'dueño & !red'              -> 1
politica 'x_desconocida | 0'         -> 0
```

**Lectura:** la acción no concedida da `0`, no `−1`. La caducidad da `0`. La ausencia de capacidad da `0`. Ampliar se rechaza. Revocar la raíz deniega a la hija. Reescribir el pasado rompe el libro. Un firmware no auditado (`0`) basta para que el dueño (`+1`) no obtenga `+1`.

### 4.2 Programas que se derivan [S]

1. **Observatorio ME:** recoge la versión del ME, el estado de AMT, el bit HAP y los avisos de seguridad aplicables, y emite un `trit` por cada invariante de la tabla 1.6. Es la auditoría convertida en cuadro de mando.
2. **Contable de permisos:** aplica la partida doble a la autoridad (ver 5.1). Detecta "descuadres", es decir, autoridad que aparece sin un origen asentado.
3. **Traductor honesto:** convierte entre formatos de credencial, declara su pérdida y pasa pruebas de ida y vuelta (invariante 9).
4. **Anillo de recuperación:** tus dispositivos (A52, PC, tableta) guardan fragmentos de forma que k de n reconstruyen la capacidad y ninguno basta por sí solo. Es la topología de anillo de 2.4 aplicada al operador `Recuperar`.
5. **Simulador de mercado de confianza:** un modelo de agentes en el que los compradores pagan según la señal de auditoría. Serviría para medir cuánto valor destruye la opacidad del 1.3.

---

## Parte V — Sueños (marcados íntegramente como [S])

### 5.1 La partida doble del permiso

Cada concesión de autoridad es un asiento: **débito** en la cuenta de capacidad del receptor y **crédito** en la cuenta de exposición del emisor. El balance de comprobación de todo el sistema debe sumar cero. La *no amplificación* deja de ser una regla y se vuelve una **ley de conservación**: la autoridad no se crea ni se destruye, solo se transfiere y se atenúa. Revocar es un **contraasiento**, no un borrado, igual que en contabilidad no se tachan los libros: se compensan. El invariante 6 de tu repositorio ya decía esto con otras palabras.

### 5.2 El token (x/∞)·Iteración

Leído como dinámica: cada iteración concede una fracción infinitesimal de autoridad, `x/∞ → 0`. Ninguna autoridad es permanente por sí misma; tiende a la indeterminación salvo que se renueve en cada iteración. Es una **autoridad con decaimiento**, lo contrario del ME, cuyo privilegio es perpetuo y no renovado. El `0` es el atractor del sistema, y el `+1` es un estado que hay que sostener activamente con consentimiento.

### 5.3 Un ME ternario

Un coprocesador de gestión que nace en `0` y tiene un interruptor físico de tres posiciones: `−1` (inerte), `0` (solo arranque) y `+1` (gestión remota con vigencia y libro visible para el dueño). Como Setun, pero para la soberanía en vez de para la aritmética.

### 5.4 Base 2 frente a base 3

La base 2 obliga a codificar "no sé" como un valor más de las dos opciones, y por eso las arquitecturas binarias tienden a colapsar la indeterminación en `sí` o `no`. La base ternaria equilibrada le da al `0` un lugar físico. Cada vez que un sistema binario decide "por defecto, permitir", comete el error que tu estado ternario prohíbe.

---

## Parte VI — Límites y próximos pasos

- **[I]** Nada de lo anterior es seguro en sentido criptográfico. El prototipo usa `id()` de Python para la revocación y un hash truncado: es pedagógico.
- **[I]** El análisis económico usa marcos estándar (Akerlof, Darby y Karni, NIC 37), pero sin datos de Intel. Las cifras son ilustrativas.
- **[I] Próximos pasos posibles:**
  1. Formalizar los operadores de `entidades-estados-operadores.md` con dominio, codominio, precondiciones y fallos, usando los axiomas A1–A6.
  2. Definir el primer esquema serializable de `Permiso` en `schemas/`, con casos válidos, inválidos y de pérdida.
  3. Decidir si TRÍA se convierte en la "cápsula ejecutable mínima" que el README exige antes de crear `src/`.
  4. Ejecutar el Observatorio ME en modo solo lectura sobre un equipo real del propietario.
