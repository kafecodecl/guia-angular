# guia-angular
Habitos de programacion en nuevas versiones de Angular

¿Por qué esta nueva guía?
Angular ya no es el de antes: pasamos de NgModule a standalone components, metimos signals, y casi que ya nada parece 2016.
Aprendimos en la trinchera: después de ver miles de líneas de código Angular por todo el mundo, teníamos datos de sobra para afinar y mejorar la guía.
El ecosistema web cambió: la tendencia a usar archivos únicos (como en Vue o Svelte) y plantillas en línea se ha disparado. Era hora de hacer ajustes.

Cambios principales
1. Reducir “consejos genéricos” que no son estrictamente de Angular
La guía anterior incluía recomendaciones como limitar archivos a 400 líneas o ser “DRY”. Aunque son consejos valiosos, no están relacionados con Angular en concreto. Hay recursos de sobra fuera de la doc oficial que hablan de esto. La nueva guía se concentra en lo verdaderamente Angular.

2. Enfocar las mejores prácticas de Angular en la documentación oficial
El viejo Style Guide metía mucha práctica recomendada sobre qué, cómo y cuándo usar ciertos patrones. Ahora, eso se moverá (o se movió) a la documentación pertinente. El Style Guide, desde este punto, se centrará más en estilo de código.

3. Renombrar clases y archivos (chau sufijos “service”, “component”, etc.)
La recomendación de nombrar nuestros archivos y clases con sufijos como *.component.ts o *.service.ts ha sido eliminada (con dos excepciones: Pipes y NgModules). El razonamiento:

Clases más descriptivas: un nombre como UserDataClient cuenta mucho más que UserService.
Menos ruido: aprender Angular no debería ser una lección de sufijos por doquier.

Excepciones:

NgModules (aunque ahora se recomienda usar componentes standalone, si usas módulos, estos seguirán teniendo sufijo *-module.ts).
Pipes (p.e. date-pipe.ts) siguen indicando el propósito de transformación de datos.

4. Fuera (casi) todas las recomendaciones de NgModule
Se recomienda trabajar con standalone components y providedIn: 'root' para servicios, así que la orientación previa sobre organizar módulos pierde fuerza. Si sigues usando módulos, está bien, pero la guía nueva no los cubre en detalle.

5. No obligar a separar la plantilla en un archivo .html
No es mandatorio extraer la plantilla a un archivo HTML si tu componente crece más de tres líneas. Usa lo que te resulte cómodo: inline template, single-file components o plantillas separadas. Angular funciona igual.

6. Cambios en @HostBinding y @HostListener
La nueva guía prefiere la propiedad host en lugar de los decoradores @HostBinding y @HostListener.

Esto facilita leer y mantener atributos como class, role o tabindex de manera centralizada.
Evita métodos triviales solo para un binding.
Se alinea mejor con la dirección hacia APIs basadas en funciones, como signals.

7. Sin prefijo on para métodos de eventos
Se acabó el onClick(), onSubmit(), etc. Mejor llama tus funciones por lo que hacen (save(), loadData(), etc.), no por el momento en el que se invocan. Vale más un buen nombre descriptivo.

8. Pequeños ajustes de nomenclatura y coordinación con herramientas
Tras los comentarios de la comunidad, se añaden dos recomendaciones nuevas:

Archivos HTML específicos de Angular terminen en .ng.html
Archivos TypeScript que importen desde @angular/core usen .ng.ts (p.e., my-component.ng.ts)

Así, herramientas como prettier o plugins de Vite pueden identificar rápidamente archivos de Angular.

Ejemplo práctico: mini aplicación con las nuevas recomendaciones
Imaginemos un caso simplificado de un componente UserProfile que use un servicio UserDataClient y un pipe UserAgePipe.

Estructura de archivos
src/
  app/
    user-profile.ng.ts         // Componente standalone de Angular
    user-data-client.ng.ts     // Servicio, sin sufijo 'service'
    user-age-pipe.ng.ts        // Pipe, mantiene sufijo 'Pipe'
    main.ts                    // Bootstrap 
user-data-client.ng.ts
import { Injectable } from '@angular/core';

// No se llama "UserService", sino algo más descriptivo
@Injectable({
  providedIn: 'root',
})
export class UserDataClient {
  private users = [{ name: 'Alice', age: 28 }, { name: 'Bob', age: 34 }];

  getUsers() {
    return this.users;
  }
} 
user-age-pipe.ng.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'userAge'
})
export class UserAgePipe implements PipeTransform {
  transform(age: number): string {
    return age > 30 ? 'Mayor de 30' : 'Menor o igual a 30';
  }
} 
user-profile.ng.ts
import { Component, inject } from '@angular/core';
import { UserDataClient } from './user-data-client.ng';
import { UserAgePipe } from './user-age-pipe.ng'; // Ejemplo de pipe

@Component({
  standalone: true,
  imports: [UserAgePipe],  // Importamos el pipe
  // Sin templateUrl, uso inline template por comodidad 
  // (ahora no es mala práctica).
  template: `
    <section class="user-profile">
      <h1>Lista de Usuarios</h1>
      <ul>
        <li *ngFor="let user of users">
          {{ user.name }} ({{ user.age | userAge }})
        </li>
      </ul>
      <button (click)="refresh()">Refrescar Lista</button>
    </section>
  `,
  // Se prefiere 'host' en lugar de @HostBinding y @HostListener
  host: {
    class: 'user-profile-container',
    '[attr.tabindex]': '0',
    '(keydown)': 'handleKeydown($event)'
  }
})
export class UserProfile {
  // Inyección de dependencias con la nueva sintaxis
  private userDataClient = inject(UserDataClient);

  get users() {
    return this.userDataClient.getUsers();
  }

  refresh() {
    console.log('Refrescando la lista de usuarios...');
    // Lógica para actualizar la lista en caso real
  }

  handleKeydown(event: KeyboardEvent) {
    if (event.key === 'Enter') {
      console.log('Enter detectado');
    }
  }
} 
main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { UserProfile } from './user-profile.ng';

bootstrapApplication(UserProfile)
  .catch(err => console.error(err)); 
Con esto, ya tenemos un ejemplo de aplicación Angular donde:

No usamos el sufijo Component ni Service.
Hemos mantenido el sufijo Pipe donde es relevante.
Aprovechamos la propiedad host en vez de @HostBinding/@HostListener.
Nombramos nuestros métodos de evento (refresh, handleKeydown) en función de su propósito, no con prefijos on.

Conclusión y próximos pasos
Feedback: La comunidad ha recibido en general de manera positiva los cambios, especialmente la simplificación y la eliminación de sufijos.
Herramientas: El equipo de Angular coordina con las herramientas de la comunidad (como angular-eslint) para agregar soporte a la nueva guía.
Refactorización opcional: Planean proveer schematics para facilitar la transición, pero será algo opt-in.
Versión: Esto no llegará en la v19 de Angular, pero sí en versiones siguientes.

Este cambio es grande, y es normal echar de menos las viejas costumbres. Pero recuerda que la Style Guide es una recomendación, no un dogma inquebrantable. Si prefieres mantener tu estilo, Angular seguirá funcionando sin problemas. Sin embargo, si buscas un approach más moderno (y un poco más ligero), estas guías te ayudarán a limpiar el código y alinearte con las mejoras que vienen en el ecosistema web.
