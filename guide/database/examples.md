# Examples

we needz them. lots of them.

These examples should be pretty thorough. 
Still need to show `having` 


## Pagination and search/filter

In this example, we loop through an array of whitelisted input fields and for each allowed non-empty value we add it to the search query. We make a clone of the query and then execute that query to count the total number of results. The count is then passed to the [Pagination](../pagination) class to determine the search offset. The second to last line searches with Pagination's items_per_page  and offset values to return a page of results based on the current page the user is on.

	$query = DB::select()->from('users');
	
	//only search for these fields
	$form_inputs = array('first_name', 'last_name', 'email');
	foreach ($form_inputs as $name) 
	{
		$value = Arr::get($_GET, $name, FALSE);
		if ($value !== FALSE AND $value != '')
		{
			$query->where($name, 'like', '%'.$value.'%');
		}
	}
	
	//copy the query & execute it
	$pagination_query = clone $query;
	$count = $pagination_query->select('COUNT("*") AS mycount')->execute()->get('mycount');
	
	//pass the total item count to Pagination
	$config = Kohana::config('pagination');
	$pagination = Pagination::factory(array(
		'total_items' => $count,
		'current_page'   => array('source' => 'route', 'key' => 'page'), 
		'total_items'    => 0,
		'items_per_page' => 20,
		'view'           => 'pagination/pretty',
		'auto_hide'      => TRUE,
	));
	$page_links = $pagination->render();
	
	//search for results starting at the offset calculated by the Pagination class
	$query->order_by('last_name', 'asc')->order_by('first_name', 'asc')->limit($pagination->items_per_page)->offset($pagination->offset);
	$results = $query->execute()->as_array();


## Having 