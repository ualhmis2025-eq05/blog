+++
date = '2025-04-02T14:10:08+02:00'
draft = true
title = 'Formulario de Contacto'
+++

<div class="contact-form">
  <form action="https://formspree.io/f/xeoarezv" method="POST">
    <label>Nombre:</label><br>
    <input type="text" name="nombre" required><br>

    <label>Email:</label><br>
    <input type="email" name="email" required><br>

    <label>Asunto:</label><br>
    <input type="text" name="asunto" required><br>

    <label>Mensaje:</label><br>
    <textarea name="mensaje" rows="5" required></textarea><br>

    <button type="submit">Enviar</button>
  </form>
</div>