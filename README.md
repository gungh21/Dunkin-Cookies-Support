import net.dv8tion.jda.api.EmbedBuilder;
import net.dv8tion.jda.api.JDABuilder;
import net.dv8tion.jda.api.entities.User;
import net.dv8tion.jda.api.events.message.MessageReceivedEvent;
import net.dv8tion.jda.api.hooks.ListenerAdapter;
import javax.security.auth.login.LoginException;
import java.awt.Color;

public class SupportBot extends ListenerAdapter {

    // Cambia esto por tu canal de staff
    private static final String STAFF_CHANNEL_ID = "ID_AQUI";

    public static void main(String[] args) throws LoginException {
        JDABuilder.createDefault("TU_TOKEN_AQUI")
                .addEventListeners(new SupportBot())
                .build();
    }

    @Override
    public void onMessageReceived(MessageReceivedEvent event) {

        // Si es el bot, ignorar
        if (event.getAuthor().isBot()) return;

        // ===========================
        // 1. MENSAJES POR DM
        // ===========================
        if (event.isFromPrivate()) {
            String userMessage = event.getMessage().getContentRaw();
            User user = event.getAuthor();

            // Respuesta automática al usuario
            user.openPrivateChannel().queue(channel -> {
                channel.sendMessage("Your message has been sent to the staff.").queue();
            });

            // Crear embed naranja y enviarlo al staff
            EmbedBuilder embed = new EmbedBuilder();
            embed.setTitle("📨 New Support Message");
            embed.setColor(new Color(255, 127, 0)); // NARANJA
            embed.addField("User", user.getAsTag() + " (" + user.getId() + ")", false);
            embed.addField("Message", userMessage, false);
            embed.setFooter("Support System");
            embed.setTimestamp(java.time.Instant.now());

            event.getJDA().getTextChannelById(STAFF_CHANNEL_ID).sendMessageEmbeds(embed.build()).queue();
        }


        // ===========================
        // 2. COMANDO PARA QUE STAFF RESPONDA
        // ===========================
        if (!event.isFromGuild()) return;

        String msg = event.getMessage().getContentRaw();

        if (msg.startsWith("!reply")) {
            String[] args = msg.split(" ", 3);

            if (args.length < 3) {
                event.getChannel().sendMessage("Uso correcto: `!reply <userId> <mensaje>`").queue();
                return;
            }

            String userId = args[1];
            String reply = args[2];

            event.getJDA().retrieveUserById(userId).queue(
                    user -> user.openPrivateChannel().queue(
                            channel -> {
                                channel.sendMessage("📩 **Staff Reply:**\n" + reply).queue();
                                event.getChannel().sendMessage("Respuesta enviada correctamente.").queue();
                            }
                    ),
                    error -> event.getChannel().sendMessage("No puedo enviar mensaje a ese usuario.").queue()
            );
        }
    }
}
