// === SISTEMA OPERATIVO SERVICIO DE LIMPIEZA ===

class Zona {
  float x, y, w, h; ; // Posición y dimensiones de la zona
  Zona(float x, float y, float w, float h) {
    this.x = x; this.y = y; this.w = w; this.h = h;
  }

  PVector puntoAleatorio() {
    return new PVector(random(x, x + w), random(y, y + h));
  }

  boolean contiene(float px, float py) {
    return (px >= x && px <= x + w && py >= y && py <= y + h);
  }
}

class Limpiador {
  PVector pos;
  PVector destino;// Posición actual y objetivo
  String nombre;
  float speed = 5.0;
  PImage imagen;
  boolean enMision = false;

  Limpiador(String nombre, float x, float y, PImage imagen) {
    this.nombre = nombre;
    this.pos = new PVector(x, y);
    this.destino = pos.copy();
    this.imagen = imagen;
  }

  void asignarDestino(PVector d) {
    destino = d.copy();
    enMision = true;
  }

  void actualizarDestino() {
    if (!enMision) {
      int zonaElegida = (int)random(zonas.size());
      Zona z = zonas.get(zonaElegida);
      destino = z.puntoAleatorio();
    }
  }

  void mover() {
    PVector dir = PVector.sub(destino, pos);
    float distancia = dir.mag();

    if (distancia < 3) {
      if (enMision) {
        enMision = false;
      }
      actualizarDestino();
      return;
    }

    dir.normalize();
    dir.mult(speed);

    PVector proximaPos = PVector.add(pos, dir);

    if (estaDentroZonas(proximaPos.x, proximaPos.y)) {
      pos = proximaPos;
    } else {
      if (!enMision) actualizarDestino();
    }
  }

  boolean estaDentroZonas(float px, float py) {
    for (Zona z : zonas) {
      if (z.contiene(px, py)) return true;
    }
    return false;
  }

  void mostrar() {
    imageMode(CENTER);
    image(imagen, pos.x, pos.y, 35, 35);
    fill(0, 100, 255);
    textAlign(CENTER);
    text(nombre, pos.x, pos.y - 25);
  }
}

class Jefe {
  float x, y;
  PImage imagen;

  Jefe(PImage imagen) {
    x = 600;
    y = 440;
    this.imagen = imagen;
  }

  void mostrar() {
    imageMode(CENTER);
    image(imagen, x, y, 40, 40);
    fill(255, 0, 0);
    textAlign(CENTER);
    text("JEFE", x, y - 25);
  }
}

class Punto {
  float x, y;
  float r = 30;
  PImage imagen;

  Punto(float x, float y, PImage imagen) {
    this.x = x;
    this.y = y;
    this.imagen = imagen;
  }

  void mostrar() {
    imageMode(CENTER);
    image(imagen, x, y, r, r);
  }

  boolean estaEncima(float px, float py) {
    return dist(px, py, x, y) <= r / 2;
  }
}
////////////////////////////////////////2parte/////////////////////////////////////////
class Orden {
  Limpiador limpiador;
  String mensaje = "¿Qué trabajo hay, jefe?";
  boolean mostrarBotones = false;
  ArrayList<Punto> basurasPendientes = new ArrayList<Punto>(); // Lista de basuras pendientes
  Punto basuraActual = null;
  int panelX, panelY;
  int tiempoSinRespuesta = 0;

  int tiempoLimpiando = 0;
  boolean limpiezaFallida = false;

  Orden(Limpiador l, int x, int y) {
    this.limpiador = l;
    this.panelX = x;
    this.panelY = y;
  }

  void mostrar() {
    fill(255, 255, 240);
    stroke(0);
    rect(panelX, panelY, 220, 100, 10);
    fill(0);
    textAlign(LEFT, TOP);
    textSize(12);
    text(limpiador.nombre, panelX + 10, panelY + 5);
    text(mensaje, panelX + 10, panelY + 25);

    if (mostrarBotones) {
      fill(200, 255, 200);
      rect(panelX + 10, panelY + 70, 40, 20);
      fill(0);
      text("SI", panelX + 20, panelY + 73);

      fill(255, 200, 200);
      rect(panelX + 60, panelY + 70, 40, 20);
      fill(0);
      text("NO", panelX + 70, panelY + 73);
    }
  }

  void click(int mx, int my) {
    if (mostrarBotones) {
      if (mx >= panelX + 10 && mx <= panelX + 50 && my >= panelY + 70 && my <= panelY + 90) {
        // SI clickeado
        if (basurasPendientes.size() > 0) {
          basuraActual = basurasPendientes.get(0);
          limpiador.asignarDestino(new PVector(basuraActual.x, basuraActual.y));
          mensaje = "Voy a limpiarlo.";
          mostrarBotones = false;
          tiempoSinRespuesta = 0;
        } else {
          mensaje = "No hay basuras pendientes.";
          mostrarBotones = false;
          tiempoSinRespuesta = 0;
        }
      } else if (mx >= panelX + 60 && mx <= panelX + 100 && my >= panelY + 70 && my <= panelY + 90) {
        // NO clickeado
        mensaje = "Está bien, jefe.";
        mostrarBotones = false;
        tiempoSinRespuesta = 0;
      }
    }
  }
}

ArrayList<Limpiador> limpiadores = new ArrayList<Limpiador>();
ArrayList<Punto> puntos = new ArrayList<Punto>();
ArrayList<Zona> zonas = new ArrayList<Zona>();
Jefe jefe;
Orden ordenCarlos;
Orden ordenLuiz;
Orden ordenAna;
Limpiador ana;

int botonX = 50;
int botonY = 700;
int botonW = 140;
int botonH = 35;

int tiempoSinRespuestaAna = 0;
boolean anaLimpiando = false;
boolean anaTermino = false;
Punto basuraActualAna = null;

PImage imgAna, imgLuiz, imgCarlos, imgJefe, imgBasura, imgOficina1, imgOficina2, imgOficina3, imgPasillo, imgJefeCuarto, imgFondo;
int tiempoJuego = 0;
int basuraRecogida = 0;

void setup() {
  size(1200, 800);
  imgAna = loadImage("ana.png");
  imgLuiz = loadImage("luiz.png");
  imgCarlos = loadImage("carlos.png");
  imgJefe = loadImage("jefe.png");
  imgBasura = loadImage("Basura.png");
  imgOficina1 = loadImage("Oficina1.jpg");
  imgOficina2 = loadImage("Oficina2.jpg");
  imgOficina3 = loadImage("Oficina3.jpg");
  imgPasillo = loadImage("Pasillo.jpg");
  imgJefeCuarto = loadImage("Jefe.jpg");
  imgFondo = loadImage("Fondo.png");

  zonas.add(new Zona(150, 100, 250, 150));
  zonas.add(new Zona(475, 100, 250, 150));
  zonas.add(new Zona(800, 100, 250, 150));
  zonas.add(new Zona(150, 250, 900, 100));

  limpiadores.add(new Limpiador("Ana", 200, 200, imgAna));
  limpiadores.add(new Limpiador("Luiz", 600, 300, imgLuiz));
  limpiadores.add(new Limpiador("Carlos", 850, 350, imgCarlos));
  
  ana = new Limpiador("Ana", 200, 600, imgAna);
  limpiadores.add(ana);
  ordenAna = new Orden(ana, 50, 620);  // Ubicación: debajo del pasillo, lado izquierdo

  for (Limpiador l : limpiadores) {
    l.actualizarDestino();
  }

  jefe = new Jefe(imgJefe);
  ordenCarlos = new Orden(limpiadores.get(2), 950, 380);
  ordenLuiz = new Orden(limpiadores.get(1), 950, 490);
}

//////////////////VOID DRAW////////////////////
void draw() {
  background(240);
  imageMode(CORNER);
  image(imgFondo, 0, 0, width, height);
  drawPlano();

  for (Punto p : puntos) p.mostrar();
  jefe.mostrar();

  for (Limpiador l : limpiadores) {
    l.mover();
    l.mostrar();
  }

  ordenCarlos.mostrar();
  ordenLuiz.mostrar();
  ordenAna.mostrar();

  verificarLimpieza(ordenCarlos);
  verificarLimpieza(ordenLuiz);
  gestionarOrden(ordenCarlos);
  gestionarOrden(ordenLuiz);

 dibujarBotonReset();
 
  tiempoJuego = (int)(millis() / 1000);  // segundos desde inicio

  fill(255, 255, 240);
  stroke(0);
  rect(width - 220, 20, 200, 50, 10);
  fill(0);
  textAlign(LEFT, CENTER);
  textSize(14);
  text("Basura recogida: " + basuraRecogida, width - 210, 45);

  int horasJuego = tiempoJuego % 24;
  fill(255, 255, 240);
  stroke(0);
  rect(20, 20, 180, 50, 10);
  fill(0);
  textAlign(CENTER, CENTER);
  textSize(14);
  text("Reloj de juego", 20 + 180 / 2, 20 + 15);
  text(horasJuego + ":00", 20 + 180 / 2, 20 + 40);
}

void dibujarBotonReset() {
  fill(0, 180, 0); // verde fuerte
  stroke(0);
  rect(botonX, botonY, botonW, botonH, 10);
  fill(255); // texto blanco para mejor contraste
  textAlign(CENTER, CENTER);
  textSize(14);
  text("RESET", botonX + botonW / 2, botonY + botonH / 2);
}


void verificarLimpieza(Orden orden) {
  if (orden.basuraActual != null &&
    dist(orden.limpiador.pos.x, orden.limpiador.pos.y,
         orden.basuraActual.x, orden.basuraActual.y) < 15) {
    puntos.remove(orden.basuraActual);
    orden.basurasPendientes.remove(orden.basuraActual);
    orden.mensaje = "¡Limpieza completada!";
    basuraRecogida++;
    orden.basuraActual = null;
    orden.limpiador.enMision = false;
    orden.tiempoLimpiando = 0;
    orden.limpiezaFallida = false;

    if (orden.basurasPendientes.size() > 0) {
      orden.basuraActual = orden.basurasPendientes.get(0);
      orden.limpiador.asignarDestino(new PVector(orden.basuraActual.x, orden.basuraActual.y));
      orden.mensaje = "Voy a limpiar otra basura.";
      orden.limpiador.enMision = true;
    } else {
      orden.mensaje = "No hay más basuras pendientes.";
    }
  }
}

void limpiarConAna() {
  if (puntos.size() > 0) {
    puntos.remove(0);
  } else {
    anaLimpiando = false;
    anaTermino = true;
  }
}

void gestionarOrden(Orden orden) {
  if (orden.mostrarBotones) {
    orden.tiempoSinRespuesta++;

    if (orden.tiempoSinRespuesta == 600) {
      orden.mensaje = "Perdón jefecito, ¿está despierto?";
    } else if (orden.tiempoSinRespuesta == 900) {
      orden.mensaje = "Hay basura en la zona. ¿Desea limpiarla?";
      orden.mostrarBotones = true;
      orden.tiempoSinRespuesta = 0;
    }
  } else {
    if (orden.basuraActual != null && orden.limpiador.enMision) {
      orden.tiempoLimpiando++;
      if (orden.tiempoLimpiando > 600 && !orden.limpiezaFallida) {
        orden.mensaje = "Perdón jefecito, no logré terminar la chamba";
        orden.limpiezaFallida = true;
      }
    } else {
      orden.tiempoLimpiando = 0;
      orden.limpiezaFallida = false;
    }
  }

  if (!ordenCarlos.limpiador.enMision && !ordenLuiz.limpiador.enMision
      && ordenCarlos.mostrarBotones && ordenLuiz.mostrarBotones) {
    tiempoSinRespuestaAna++;
  } else {
    tiempoSinRespuestaAna = 0;
  }

  if (tiempoSinRespuestaAna >= 120 && !anaLimpiando && !anaTermino) {
    anaLimpiando = true;
    tiempoSinRespuestaAna = 0;
    basuraActualAna = puntos.get(0);
    limpiadores.get(0).asignarDestino(new PVector(basuraActualAna.x, basuraActualAna.y));
  }

  if (anaLimpiando && !anaTermino) {
    limpiarConAna();
  }

}void gestionarAna() {
  Limpiador ana = limpiadores.get(0);  // Ana es la primera en la lista

  if (basuraActualAna != null && dist(ana.pos.x, ana.pos.y, basuraActualAna.x, basuraActualAna.y) < 15) {
    puntos.remove(basuraActualAna);  // Elimina basura del mapa
    basuraActualAna = null;

    if (puntos.size() > 0) {
      // Queda más basura, ir a la siguiente
      basuraActualAna = puntos.get(0);
      ana.asignarDestino(new PVector(basuraActualAna.x, basuraActualAna.y));
    } else {
      // No queda más basura
      anaLimpiando = false;
      anaTermino = true;
    }
  }
}
void drawPlano() {
  stroke(20);
  strokeWeight(4);

  drawFondoOficina1();
  drawFondoOficina2();
  drawFondoOficina3();
  drawFondoPasillo();
  drawFondoCuartoJefe();

  drawPlanoOficina(150, 100, 265, 150, "Oficina 1");
  drawPlanoOficina(475, 100, 250, 150, "Oficina 2");
  drawPlanoOficina(800, 100, 250, 150, "Oficina 3");

  drawPlanoPasilloYJefe(150, 250, 900, 280);
}

void drawPlanoOficina(int x, int y, int w, int h, String nombre) {
  int puerta = w / 3;
  line(x, y, x + w, y);
  line(x, y, x, y + h);
  line(x + w, y, x + w, y + h);
  line(x, y + h, x + puerta, y + h);
  line(x + 2 * puerta, y + h, x + w, y + h);

  fill(0);
  textAlign(CENTER, CENTER);
  textSize(16);
  text(nombre, x + w / 2, y + h / 2);
}

void drawPlanoPasilloYJefe(int x, int y, int w, int hTotal) {
  int pasilloH = 100;
  int jefeH = hTotal - pasilloH;

  int[] oficinaX = {150, 475, 800};
  int oficinaW = 250;
  int puertaW = oficinaW / 3;

  int px = x;
  for (int i = 0; i < 3; i++) {
    int puertaInicio = oficinaX[i] + oficinaW / 3;
    line(px, y, puertaInicio, y);
    px = puertaInicio + puertaW;
    fill(255, 255, 240);
stroke(0);
rect(150, 360, 220, 50, 10);  // Ajusta posición si quieres
fill(0);
textAlign(LEFT, TOP);
textSize(12);

if (anaLimpiando) {
  text("Ana: VOY A TERMINAR EL TRABAJO JEFESITO", 155, 365);
} else if (anaTermino) {
  text("Ana: TODO LIMPIO JEFESITO", 155, 365);
}

  }
  line(px, y, x + w, y);

  line(x, y, x, y + pasilloH);
  line(x + w, y, x + w, y + pasilloH);

  fill(0);
  textAlign(CENTER, CENTER);
  textSize(18);
  text("Pasillo", x + w / 2, y + pasilloH / 2);

  int jefeX = x + w / 3;
  int jefeW = w / 3;
  int jefeY = y + pasilloH;
  int puertaJefe = jefeW / 3;

  line(jefeX, jefeY, jefeX, jefeY + jefeH);
  line(jefeX + jefeW, jefeY, jefeX + jefeW, jefeY + jefeH);
  line(jefeX, jefeY + jefeH, jefeX + jefeW, jefeY + jefeH);
  line(jefeX, jefeY, jefeX + (jefeW - puertaJefe) / 2, jefeY);
  line(jefeX + (jefeW + puertaJefe) / 2, jefeY, jefeX + jefeW, jefeY);

  fill(0);
  textSize(16);
  text("", x + w / 2, jefeY + jefeH / 2);

  line(x, y + pasilloH, jefeX, jefeY);
  line(x + w, y + pasilloH, jefeX + jefeW, jefeY);
}

void drawFondoOficina1() {
  int x = 150;
  int y = 100;
  int w = 270;
  int h = 150;
  imageMode(CORNER);
  image(imgOficina1, x + 4, y + 4, w - 8, h - 8);
}

void drawFondoOficina2() {
  int x = 475;
  int y = 100;
  int w = 250;
  int h = 150;
  imageMode(CORNER);
  image(imgOficina2, x + 4, y + 4, w - 8, h - 8);
}

void drawFondoOficina3() {
  int x = 800;
  int y = 100;
  int w = 250;
  int h = 150;
  imageMode(CORNER);
  image(imgOficina3, x + 4, y + 4, w - 8, h - 8);
}

void drawFondoPasillo() {
  int x = 150;
  int y = 250;
  int w = 900;
  int h = 100;

  imageMode(CORNER);
  image(imgPasillo, x + 4, y + 4, w - 8, h - 8);
}

void drawFondoCuartoJefe() {
  int x = 150;
  int y = 250;
  int w = 900;
  int pasilloH = 100;
  int jefeX = x + w / 3;
  int jefeW = w / 3;
  int jefeY = y + pasilloH;
  int jefeH = 180;

  imageMode(CORNER);
  image(imgJefeCuarto, jefeX + 4, jefeY + 4, jefeW - 8, jefeH - 8);
}

void mousePressed() {
  
   if (mouseButton == LEFT) {
    if (mouseX >= 540 && mouseX <= 680 && mouseY >= 70 && mouseY <= 100) {
      resetSistema();
      return; // para evitar más procesamiento este clic
    }
  }
  if (mouseButton == RIGHT) {
    boolean eliminado = false;
    for (int i = puntos.size() - 1; i >= 0; i--) {
      if (puntos.get(i).estaEncima(mouseX, mouseY)) {
        puntos.remove(i);
        eliminado = true;
        break;
      }
    }
    if (!eliminado) {
      Punto nueva = new Punto(mouseX, mouseY, imgBasura);
      puntos.add(nueva);
      for (int i = 0; i < zonas.size(); i++) {
        if (zonas.get(i).contiene(mouseX, mouseY)) {
          String nombreZona = i == 0 ? "Oficina 1" : i == 1 ? "Oficina 2" : i == 2 ? "Oficina 3" : "Pasillo";

          if (!ordenCarlos.basurasPendientes.contains(nueva)) ordenCarlos.basurasPendientes.add(nueva);
          if (!ordenLuiz.basurasPendientes.contains(nueva)) ordenLuiz.basurasPendientes.add(nueva);

          if (!ordenCarlos.limpiador.enMision) {
            ordenCarlos.mensaje = "Hay basura en " + nombreZona + ". ¿Puedo limpiarla?";
            ordenCarlos.mostrarBotones = true;
          }
          if (!ordenLuiz.limpiador.enMision) {
            ordenLuiz.mensaje = "Hay basura en " + nombreZona + ". ¿Puedo ir también?";
            ordenLuiz.mostrarBotones = true;
          }
          break;
        }
      }
    }
  } else if (mouseButton == LEFT) {
    ordenCarlos.click(mouseX, mouseY);
    ordenLuiz.click(mouseX, mouseY);
  }
}
void resetSistema() {
  // Reiniciar limpiadores: posiciones y destinos
  for (Limpiador l : limpiadores) {
    l.pos = new PVector(random(width/4, width/2), random(height/2, height*3/4));
    l.destino = l.pos.copy();
    l.enMision = false;
  }

  // Reiniciar las órdenes
  ordenCarlos.mensaje = "¿Qué chamba hay, jefe?";
  ordenCarlos.mostrarBotones = false;
  ordenCarlos.basurasPendientes.clear();
  ordenCarlos.basuraActual = null;
  ordenCarlos.tiempoSinRespuesta = 0;
  ordenCarlos.tiempoLimpiando = 0;
  ordenCarlos.limpiezaFallida = false;

  ordenLuiz.mensaje = "¿Qué Laburo hay, jefe?";
  ordenLuiz.mostrarBotones = false;
  ordenLuiz.basurasPendientes.clear();
  ordenLuiz.basuraActual = null;
  ordenLuiz.tiempoSinRespuesta = 0;
  ordenLuiz.tiempoLimpiando = 0;
  ordenLuiz.limpiezaFallida = false;

  ordenAna.mensaje = "¿Qué trabajo hay, jefe?";
  ordenAna.mostrarBotones = false;
  ordenAna.basurasPendientes.clear();
  ordenAna.basuraActual = null;
  ordenAna.tiempoSinRespuesta = 0;
  ordenAna.tiempoLimpiando = 0;
  ordenAna.limpiezaFallida = false;

  puntos.clear(); // Limpiar todas las basuras en pantalla

  // Reiniciar estados Ana
  anaLimpiando = false;
  anaTermino = false;
  tiempoSinRespuestaAna = 0;
}
