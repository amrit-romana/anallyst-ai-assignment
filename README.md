I didn't had the wordpress premium so couldn't do the possible assignment precisely but I have coded the sample exactlty into html and css

 -----------------------------------------------------------
 
 
To create a plugin :-
Create a new plugin
Create a new plugin by creating a new directory in the WordPress plugins directory (/wp-content/plugins/) and adding a new PHP file in it. The name of the PHP file should match the name of the directory.

Create a database table
Use the following code to create a database table on activation of the plugin:

function create_table() {
    global $wpdb;
    $table_name = $wpdb->prefix . 'ticket_bookings';
    $sql = "CREATE TABLE IF NOT EXISTS $table_name (
            id mediumint(9) NOT NULL AUTO_INCREMENT,
            ticket_number mediumint(9) NOT NULL,
            PRIMARY KEY (id)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci";
    require_once(ABSPATH . 'wp-admin/includes/upgrade.php');
    dbDelta($sql);
}

register_activation_hook( __FILE__, 'create_table' );

Add 100 fields to the database table
Use the following code to add 100 fields with 0 value to the database table:

function add_fields_to_table() {
    global $wpdb;
    $table_name = $wpdb->prefix . 'ticket_bookings';
    for ($i=1; $i<=100; $i++) {
        $wpdb->insert($table_name, array('ticket_number' => $i));
    }
}

register_activation_hook( __FILE__, 'add_fields_to_table' );


Create shortcode
Use the following code to create the shortcode [ticket_book_cf7]:

function ticket_book_cf7_shortcode() {
    global $wpdb;
    $table_name = $wpdb->prefix . 'ticket_bookings';
    $booked_tickets = $wpdb->get_col("SELECT ticket_number FROM $table_name");
    $disabled_tickets = implode(',', $booked_tickets);
    $html = '<div class="ticket-booking-cf7">';
    $html .= do_shortcode('[contact-form-7 id="1234" html_id="my-form"]');
    $html .= '<script>document.addEventListener( \'wpcf7submit\', function( event ) {
                var submitted_ticket = parseInt(event.detail.inputs[0].value);
                var disabled_tickets = ['.$disabled_tickets.'];
                if(disabled_tickets.includes(submitted_ticket)) {
                    var submitButton = document.querySelector(\'[type="submit"]\');
                    submitButton.disabled = true;
                    submitButton.value = \'Ticket already booked\';
                } else {
                    jQuery.ajax({
                        url: \''.admin_url( 'admin-ajax.php' ).'\',
                        type: \'POST\',
                        data: {\'action\': \'ticket_book_cf7_ajax\', \'ticket_number\': submitted_ticket},
                        success: function() {
                            console.log(\'Ticket booked successfully\');
                        },
                        error: function() {
                            console.log(\'Ticket booking failed\');
                        }
                    });
                }
            }, false );</script>';
    $html .= '</div>';
    return $html;
}

add_shortcode( 'ticket_book_cf7', 'ticket_book_cf7_shortcode' );

Add shortcode to contact form
Add the shortcode [ticket_book_cf7] before the submit button in the contact form.

Add frontend functionality
Add the following code to display 100 checkboxes in the contact form:


<input type="checkbox" name="ticket_number[]" value="<?php echo $i; ?>" <?php if(in_array($i, $booked_tickets)) {echo 'disabled';} ?>>
``


