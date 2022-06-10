# ops_file_format
 The Open PV Site file format


This open and extendible file format can be used to share geo-information of solar installations between, designers, asset owners and inspection companies.
Until now, at the absence of a commonly used file format, information of solar installations, specifically the position and name of PV modules is often manually traced by inspection companies using provided CAD drawings.
These drawings generally do not follow common design principles and therefore dont allow for straign forward automated parsing.
For every project of a measurement company many hours are wasted in this work that could be saved if all active groups would share the site information in a standardised file format.

Purpose of this project is providing the industry with a free an easy to use tool to read to, write from and visualize data of this format.
Code will be provided in multiple common languages including Python, Java, C++.

The following text describes this binary file format. For implementations in different languages, see directory 'code'.    



*byte order is BIG ENDIAN*



## Type Definitions

| Name        | Description  | Size(bytes) |
| ------------- |:----------:|-----:| 
| u8      | Unsigned integer | 1 |
| u16     | Unsigned integer |  2 |
| u32     | Unsigned integer |   4 |
| bits8     | boolean array saved in 8 bits |   1 |
| bits16     | boolean array saved in 16 bits |   2 |
| f64     | Floating point number |   8 |
| str8     | String size[u8], String - utf8 formatted |   variable |
| point2_u8     | 2d point of u8 e.g. p0.x=2,p0.y=4,p1.x=34,p1.y=45 => {2,4,34,45} |   2 |
| point2_f64     | same but using [f64] for x,y | 16    |
| quad_f64     | array of 4 points using [f64]. points include z parameter if flag <points_contain_height>==1, then {p0.x,p0.y,p0.z,...,p3.x,p3.y,p3.z} |   64 or  96|
| quad2_f64     | same as above but always using 2d points |   64 |



### Added '*' (for u8, u16, u23, str8)

[str8] can only store strings up to a length of 255 characters. 
For longer strings, the first byte is set to 255.
In this case the next 4 bytes are used to describe the size as [u32]

E.g. 
Short string:
{5,'He','e','l','l,'o'} 
Long string:
{255,0,0,255,1,'H',...} -> read this as size 256



## File Layout of version 0.1

```c
file_type[3 bytes] => {'O','P','S'}
file_type_version[u16] => {0,1} 
file_checksum[u16] => CRC16 - to ensure data(starting from next byte) is not corrupted
file_author[str8]
file_creation_date[u32] => seconds elapsed since 00:00 hours, Jan 1, 1970 UTC (i.e., a unix timestamp)
flags[bits16] => {1010000000000000}
	bit 0: points_are_geo_position [true: string corners are longitude,latitude[degrees], false: points are scaled to arbitrary units]
	bit 1: points_contain_height [true: points are [longitude,latitude,height(meters)], false: points are [longitude,latitude]
	bit 2: point_height_is_altitude [true: height is height above sea level]
	bit 3: file_contains_module_interconnection
	bit 4: file_contains_prism_objects
	bit 5: file_contains_poly_objects
	bit 6: file_contains_surface
	bit 7: file_contains_project_details
	bit 8: file_contains_custom_data
	bit 9-15: T.B.A.

// dict: {name: region}
n_regions[u8*] => {1}
	
	// first region
	region_name[str8*] => {10,'G','r','e','e','f','i','e','l','d',1}
	region_corners[quad(xy)_f64]
	n_conn_points[u8*]
		
		// first connection point
		n_string_groups[u8*]
		
			// first string groug e.g. inverter or combiner box
			n_strings[u8*]
			
				
				// first string
				string_name[str8*]
				n_rows[u8*]
					// first string row
					n_modules[point2_8]
					row_corners[quad64]
					...
					
				if(file_contains_module_interconnection)
					
					n_parallel_strings[u8*]
					for _ in range(n_all_modules_in_string):
						// list connection order from + to - 
						next_module[point2_u8*]
				...
			...
		...
		
	if(file_contains_prism_objects)
		n_prism_objects[u16*]
			// first prism
			prism_name[str8*]
			prism_height[f64]
			n_corners[u8*]
				corner[point2_f64]
				...
			...
			
			
	if(file_contains_poly_objects)
		// TODO: find format to store 3d polygons
		// containing number_of_faces,face_index_vertex_index, vertex_coordinates, ?texture_index
	
		n_poly_objects[u16*]
			// first polygon
			...
		...

	...


if(file_contains_surface)
	//TODO
if(file_contains_project_details)
	//TODO
if(file_contains_custom_data)
	buffer_size[u32]
```
